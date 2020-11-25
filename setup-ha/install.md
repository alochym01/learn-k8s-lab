# Kubernetes HA setup with Stacked ETCD
## Network Topology
## Requirements
1.  OS
    1.  Ubuntu 18
        -   Hardware requirements minimal
            -   RAM: 2G
            -   CPU: 2
    2.  docker container CE
    3.  kubernetes version 1.18
    4.  HA Proxy version 2.2 act as loadbalancer
    5.  IP Addess
        1.  HA Proxy
            -   192.168.1.2
            -   hostname: k8smaster
        2.  Master Nodes
            -   172.16.1.2/24
            -   172.16.1.3/24
            -   172.16.1.4/24
        3.  Worker Nodes
            -   172.16.1.5/24
            -   172.16.1.6/24
            -   172.16.1.7/24
2.  2 HA Proxy
    1.  `sudo apt-get install --no-install-recommends software-properties-common`
    2.  `sudo add-apt-repository ppa:vbernat/haproxy-2.2`
    3.  `sudo apt-get install haproxy=2.2.\*`
    4.  config `/etc/haproxy/haproxy.cfg`
        ```bash
        global
            log /dev/log	local0
            log /dev/log	local1 notice
            chroot /var/lib/haproxy
            stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
            stats timeout 30s
            user haproxy
            group haproxy
            daemon

            # Default SSL material locations
            ca-base /etc/ssl/certs
            crt-base /etc/ssl/private

            # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
            ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
            ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
            ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

        defaults
            log	global
            mode	tcp
            option	tcplog
            option	dontlognull
            timeout connect 5000
            timeout client  50000
            timeout server  50000
            errorfile 400 /etc/haproxy/errors/400.http
            errorfile 403 /etc/haproxy/errors/403.http
            errorfile 408 /etc/haproxy/errors/408.http
            errorfile 500 /etc/haproxy/errors/500.http
            errorfile 502 /etc/haproxy/errors/502.http
            errorfile 503 /etc/haproxy/errors/503.http
            errorfile 504 /etc/haproxy/errors/504.http

        frontend proxynode #<-- Add the following lines to bottom of file
            bind *:6443
            stats uri /proxystats
            default_backend k8sServers

        backend k8sServers
            balance roundrobin
            server FirstMaster 172.16.1.2:6443 check #<-- Edit these with your IP addresses, port, and hostname
            server SecondMaster 172.16.1.3:6443 check #<-- Comment out until ready
            server ThirdMaster 172.16.1.4:6443 check #<-- Comment out until ready

        frontend monitoring
            bind *:8404
            mode http
            option http-use-htx
            http-request use-service prometheus-exporter if { path /metrics }
            stats enable
            stats uri /stats
            stats refresh 10s
        ```
    5.  Start haproxy server - `sudo systemctl start haproxy`
3.  All kubernetes nodes
    -   Setup for all nodes
        1.  `sudo swapoff -v /swappfile`
        2.  Remove the swap file entry from the /etc/fstab file
            -   `/swapfile swap swap defaults 0 0`
            -   `sudo rm /swapfile`
        3.  Stop firewall and disabled - using for Lab
            -   `sudo systemctl stop ufw`
            -   `sudo systemctl disable ufw`
            -   `sudo reboot`
        4.  IP Tables
            -   br_netfilter module is loaded
                -   Check - `lsmod | grep br_netfilter`
                -   Add - `sudo modprobe br_netfilter`
            -   net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config
                ```bash
                cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
                    net.bridge.bridge-nf-call-ip6tables = 1
                    net.bridge.bridge-nf-call-iptables = 1
                EOF

                sudo sysctl --system
                ```
        5.  Install kubernetes and docker software
            -   `sudo apt-get update`
            -   `sudo apt-get install -y apt-transport-https ca-certificates curl gnupg2 wget software-properties-common`
            -   `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`
            -   Create kubernetes repo
                ```bash
                cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
                    deb https://apt.kubernetes.io/ kubernetes-xenial main
                EOF
                ```
            -   `sudo apt-get update`
            -   `sudo apt-get install -y kubeadm=1.18.1-00 kubelet=1.18.1-00 kubectl=1.18.1-00 vim docker.io`
            -   `sudo apt-mark hold kubelet kubeadm kubectl`
            -   Create docker daemon.json
                ```bash
                cat > /etc/docker/daemon.json <<EOF
                {
                    "exec-opts": ["native.cgroupdriver=systemd"],
                    "log-driver": "json-file",
                    "log-opts": {
                        "max-size": "100m"
                    },
                    "storage-driver": "overlay2"
                }
                EOF
                ```
            -   Start `docker` and `kubelet`
                -   `sudo systemctl enable docker`
                -   `sudo systemctl enable kubelet`
                -   `sudo systemctl start docker`
                -   `sudo systemctl start kubelet`
    -   Master 1
        1.  Create kubeadm.yaml
            ```yaml
            apiVersion: kubeadm.k8s.io/v1beta2
            kind: ClusterConfiguration
            kubernetesVersion: 1.18.1 #<-- Use the word stable for newest version
            controlPlaneEndpoint: "k8smaster:6443" #<-- Use the node alias not the IP
            networking:
                podSubnet: 10.16.0.0/16 #<-- Match the IP range from the Calico config file
            ```
        2.  Download calico cni and edit `calico.yaml` - using normal account
            -   `wget https://docs.projectcalico.org/manifests/calico.yaml`
            ```bash
            #Pod IPs will be chosen from this range. This should fall within `--cluster-cidr`
            -   name: CALICO_IPV4POOL_CIDR
                value: "10.16.0.0/16"
            ```
        3.  `kubeadm init --config=kubeadm-config.yaml --upload-certs | tee kubeadm-init.out`
        4.  setup kubectl - using normal account
            ```bash
            mkdir -p $HOME/.kube
            sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
            sudo chown $(id -u):$(id -g) $HOME/.kube/config
            kubectl apply -f calico.yaml
            ```
    -   Master 2
        1.  setup hostname point to Master 1
            ```bash
            vim /etc/hosts => set k8smaster 192.168.1.2
            ```
        2.  Create token key and certificate key
            -   certificate key
                ```bash
                kubeadm alpha certs certificate-key
                28088afa1cb5d6fc4ece5596a930ca3d2d6526f010f4d8bade7ece2fa6d2045c
                ```
            -   token key
                ```bash
                kubeadm token create --print-join-command
                kubeadm join master-1:6443 --token aeyr82.n3w6rom0g67k6y6g --discovery-token-ca-cert-hash sha256:43b1d1e08714bc52a6c3409e9c307ffae8ac1de3ef56269a8b6470dbf85a2c54
                ```
        3.  Join Master 2
            ```bash
            kubeadm join master-1:6443 --token aeyr82.n3w6rom0g67k6y6g --discovery-token-ca-cert-hash sha256:43b1d1e08714bc52a6c3409e9c307ffae8ac1de3ef56269a8b6470dbf85a2c54 --control-plane --certificate-key 28088afa1cb5d6fc4ece5596a930ca3d2d6526f010f4d8bade7ece2fa6d2045c
            ```
    -   Master 3
        1.  setup hostname point to Master 1
            ```bash
            vim /etc/hosts => set k8smaster 192.168.1.2
            ```
        2.  Join Master 3
            ```bash
            kubeadm join master-1:6443 --token aeyr82.n3w6rom0g67k6y6g --discovery-token-ca-cert-hash sha256:43b1d1e08714bc52a6c3409e9c307ffae8ac1de3ef56269a8b6470dbf85a2c54 --control-plane --certificate-key 28088afa1cb5d6fc4ece5596a930ca3d2d6526f010f4d8bade7ece2fa6d2045c
            ```
4.  All 3 worker nodes
    1.  setup hostname point to Master 1
        ```bash
        vim /etc/hosts => set k8smaster 192.168.1.2
        ```
    2.  Join Worker Node
        ```bash
        kubeadm join master-1:6443 --token aeyr82.n3w6rom0g67k6y6g --discovery-token-ca-cert-hash sha256:43b1d1e08714bc52a6c3409e9c307ffae8ac1de3ef56269a8b6470dbf85a2c54
        ```
5.  Check Installation
    -   `kubectl cluster-info`
    -   `kubectl get nodes`
