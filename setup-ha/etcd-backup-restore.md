# Lab backup and restore Single ETCD
## Check etcd version
1.  kubectl describe pod etcd-master-1 -n kube-system
    ```bash
    Name:                 etcd-master-1
    Namespace:            kube-system
    Priority:             2000000000
    Priority Class Name:  system-cluster-critical
    Node:                 master-1/192.168.2.150
    Start Time:           Thu, 19 Nov 2020 15:35:31 +0700
    Labels:               component=etcd
                          tier=control-plane
    Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.2.150:2379
                          kubernetes.io/config.hash: 728b9eec74592aa8c5af57a913935e3a
                          kubernetes.io/config.mirror: 728b9eec74592aa8c5af57a913935e3a
                          kubernetes.io/config.seen: 2020-07-02T15:15:06.060314326+07:00
                          kubernetes.io/config.source: file
    Status:               Running
    IP:                   192.168.2.150
    IPs:
    IP:           192.168.2.150
    Controlled By:  Node/master-1
    Containers:
    etcd:
        Container ID:  docker://84014414351bace4e05342e5b7606ec75820ca7f963441400d4ddb8e45da7707
        Image:         k8s.gcr.io/etcd:3.4.3-0
        Image ID:      docker-pullable://k8s.gcr.io/etcd@sha256:4afb99b4690b418ffc2ceb67e1a17376457e441c1f09ab55447f0aaf992fa646
        Port:          <none>
        Host Port:     <none>
        Command:
          etcd
          --advertise-client-urls=https://192.168.2.150:2379
          --cert-file=/etc/kubernetes/pki/etcd/server.crt
          --client-cert-auth=true
          --data-dir=/var/lib/etcd
          --initial-advertise-peer-urls=https://192.168.2.150:2380
          --initial-cluster=master-1=https://192.168.2.150:2380
          --key-file=/etc/kubernetes/pki/etcd/server.key
          --listen-client-urls=https://127.0.0.1:2379,https://192.168.2.150:2379
          --listen-metrics-urls=http://127.0.0.1:2381
          --listen-peer-urls=https://192.168.2.150:2380
          --name=master-1
          --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
          --peer-client-cert-auth=true
          --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
          --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
          --snapshot-count=10000
          --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
        State:          Running
          Started:      Thu, 19 Nov 2020 15:35:42 +0700
        Last State:     Terminated
          Reason:       Error
          Exit Code:    255
          Started:      Fri, 06 Nov 2020 15:54:01 +0700
          Finished:     Thu, 19 Nov 2020 15:35:09 +0700
        Ready:          True
        Restart Count:  8
        Liveness:       http-get http://127.0.0.1:2381/health delay=15s timeout=15s period=10s #success=1 #failure=8
        Environment:    <none>
        Mounts:
          /etc/kubernetes/pki/etcd from etcd-certs (rw)
          /var/lib/etcd from etcd-data (rw)
    Conditions:
    Type              Status
      Initialized       True
      Ready             True
      ContainersReady   True
      PodScheduled      True
    Volumes:
      etcd-certs:
        Type:          HostPath (bare host directory volume)
        Path:          /etc/kubernetes/pki/etcd
        HostPathType:  DirectoryOrCreate
      etcd-data:
        Type:          HostPath (bare host directory volume)
        Path:          /var/lib/etcd
        HostPathType:  DirectoryOrCreate
    QoS Class:         BestEffort
    Node-Selectors:    <none>
    Tolerations:       :NoExecute
    Events:            <none>
    ```
## Backup Certificates
1.  cp -r /etc/kubernetes/pki /opt/alochym/certs-backup
## Backup ETCD
1.  Create a snapshot etcd server using etcdctl cli
    1.  copy etcdctl from pod in `kube-system` namespace - `kubectl cp kube-system/etcd-master-1:/usr/local/bin/etcdctl etcd/etcdctl`
    2.  save a snapshot. if you have a cluster, should restore for 3 times and then start cluster
        ```bash
        ETCDCTL_API=3 \
        ./etcdctl snapshot save /opt/alochym/etcd-snapshot.db \
        --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/server.crt
        --key=/etc/kubernetes/pki/etcd/server.key
        ```
    3.  Test backup file
        ```bash
        root@master-1:/home/hadn/etcd# ./etcdctl --write-out=table snapshot status etcd-snapshot.db
        +----------+----------+------------+------------+
        |   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
        +----------+----------+------------+------------+
        | c180d0f6 | 17336887 |       1639 |     5.0 MB |
        +----------+----------+------------+------------+
        ```
## Restore ETCD
1.  Testing to restore
    1.  reset everything of kubernetes - `kubeadm reset -f`
    2.  copy everything backup certs to /etc/kubernetes/ - `cp -r /opt/alochym/certs-backup /etc/kubernetes/`
    3.  restore etcd server
        ```bash
        ETCDCTL_API=3 \
        ./etcdctl snapshot restore /opt/alochym/etcd-snapshot.db \
          --endpoints=https://127.0.0.1:2379 \
          --cacert=/etc/kubernetes/pki/etcd/ca.crt \
          --cert=/etc/kubernetes/pki/etcd/server.crt \
          --key-file=/etc/kubernetes/pki/etcd/server.key \
          --name=master-1 \
          --data-dir="/opt/alochym/restore" \
          --initial-cluster=master-1=https://127.0.0.1:2380 \
          --initial-cluster-token="etcd-cluster-1" \
          --initial-advertise-peer-urls=https://127.0.0.1:2380
        ```
    4.  Edit `/etc/kubernetes/manifests/etcd.yaml` to use the etcd restore folder
        ```bash
        apiVersion: v1
        kind: Pod
        metadata:
        annotations:
            kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.2.150:2379
        creationTimestamp: null
        labels:
            component: etcd
            tier: control-plane
        name: etcd
        namespace: kube-system
        spec:
        containers:
        - command:
            - etcd
            - --advertise-client-urls=https://192.168.2.150:2379
            - --cert-file=/etc/kubernetes/pki/etcd/server.crt
            - --client-cert-auth=true
            - --data-dir=/var/lib/etcd
            - --initial-advertise-peer-urls=https://192.168.2.150:2380
            - --initial-cluster=master-1=https://192.168.2.150:2380
            - --key-file=/etc/kubernetes/pki/etcd/server.key
            - --listen-client-urls=https://127.0.0.1:2379,https://192.168.2.150:2379
            - --listen-metrics-urls=http://127.0.0.1:2381
            - --listen-peer-urls=https://192.168.2.150:2380
            - --name=master-1
            - --initial-cluster-token=etcd-cluster-1
            - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
            - --peer-client-cert-auth=true
            - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
            - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
            - --snapshot-count=10000
            - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
            image: k8s.gcr.io/etcd:3.4.3-0
            imagePullPolicy: IfNotPresent
            livenessProbe:
            failureThreshold: 8
            httpGet:
                host: 127.0.0.1
                path: /health
                port: 2381
                scheme: HTTP
            initialDelaySeconds: 15
            timeoutSeconds: 15
            name: etcd
            resources: {}
            volumeMounts:
            - mountPath: /var/lib/etcd
            name: etcd-data
            - mountPath: /etc/kubernetes/pki/etcd
            name: etcd-certs
        hostNetwork: true
        priorityClassName: system-cluster-critical
        volumes:
        - hostPath:
            path: /etc/kubernetes/pki/etcd
            type: DirectoryOrCreate
            name: etcd-certs
        - hostPath:
            path: /opt/alochym/restore # modify same as restore step
            type: DirectoryOrCreate
            name: etcd-data
        status: {}
        ```
