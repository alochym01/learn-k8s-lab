# Upgrade K8s version 1.18.x to 1.19.4
## Requirements Steps
1.  Find the latest stable 1.19 version
    ```bash
    apt update
    apt-cache madison kubeadm
    kubeadm |  1.19.4-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubeadm |  1.19.3-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubeadm |  1.19.2-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubeadm |  1.19.1-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubeadm |  1.19.0-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    ....
    kubeadm |   1.6.1-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubeadm |   1.5.7-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    ```
## Upgrade Primary Control Plane
### Upgrade kubeadm
1.  `apt-mark unhold kubeadm`
2.  `apt-get update && apt-get install -y kubeadm=1.19.4-00`
3.  `apt-mark hold kubeadm`
4.  Verify kubeadm version - `kubeadm version`
### Make unscheduleable of control plane
1.  `kubectl drain master-1 --ignore-daemonsets`
    ```bash
    node/master-1 cordoned
    WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-sccp8, kube-system/kube-proxy-xxh5h
    evicting pod kube-system/calico-kube-controllers-8586758878-4z64l
    evicting pod kube-system/coredns-66bff467f8-8fdp7
    evicting pod kube-system/coredns-66bff467f8-zgxt6
    pod/calico-kube-controllers-8586758878-4z64l evicted
    pod/coredns-66bff467f8-zgxt6 evicted
    pod/coredns-66bff467f8-8fdp7 evicted
    node/master-1 evicted
    ```
2.  Check which versions are available to upgrade - `kubeadm upgrade plan`
    ```bash
    ...
    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    kubelet     2 x v1.18.1   v1.19.4

    Upgrade to the latest stable version:

    COMPONENT                 CURRENT   AVAILABLE
    kube-apiserver            v1.18.1   v1.19.4
    kube-controller-manager   v1.18.1   v1.19.4
    kube-scheduler            v1.18.1   v1.19.4
    kube-proxy                v1.18.1   v1.19.4
    CoreDNS                   1.6.7     1.7.0
    etcd                      3.4.3-0   3.4.13-0

    You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.19.4
    ```
3.  Upgrade k8s to version 1.19.4 - `kubeadm upgrade apply v1.19.4`
### Upgrade kubelet and kubectl
1.  apt-mark unhold kubelet kubectl
2.  apt-get install -y kubelet=1.19.4-00 kubectl=1.19.4-00
3.  apt-mark hold kubelet kubectl
4.  systemctl daemon-reload
5.  systemctl restart kubelet
6.  kubectl uncordon master - Now make the master available for the scheduler
7.  kubectl get node
## Upgrade Additional Control Plane
### Upgrade kubeadm
1.  `apt-mark unhold kubeadm`
2.  `apt-get update && apt-get install -y kubeadm=1.19.4-00`
3.  `apt-mark hold kubeadm`
4.  Verify kubeadm version - `kubeadm version`
### Make unscheduleable of control plane
1.  `kubectl drain master-1 --ignore-daemonsets` - Gracefully terminate all pods on a node
    ```bash
    node/master-1 cordoned
    WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-sccp8, kube-system/kube-proxy-xxh5h
    evicting pod kube-system/calico-kube-controllers-8586758878-4z64l
    evicting pod kube-system/coredns-66bff467f8-8fdp7
    evicting pod kube-system/coredns-66bff467f8-zgxt6
    pod/calico-kube-controllers-8586758878-4z64l evicted
    pod/coredns-66bff467f8-zgxt6 evicted
    pod/coredns-66bff467f8-8fdp7 evicted
    node/master-1 evicted

    root@master-1:~# kubectl get nodes --kubeconfig=/home/hadn/.kube/config
    NAME       STATUS                     ROLES    AGE    VERSION
    master-1   Ready,SchedulingDisabled   master   6d6h   v1.18.1
    worker-1   Ready                      <none>   6d6h   v1.18.1
    ```
2.  Check which versions are available to upgrade - `kubeadm upgrade plan`
    ```bash
    ...
    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    kubelet     2 x v1.18.1   v1.19.4

    Upgrade to the latest stable version:

    COMPONENT                 CURRENT   AVAILABLE
    kube-apiserver            v1.18.1   v1.19.4
    kube-controller-manager   v1.18.1   v1.19.4
    kube-scheduler            v1.18.1   v1.19.4
    kube-proxy                v1.18.1   v1.19.4
    CoreDNS                   1.6.7     1.7.0
    etcd                      3.4.3-0   3.4.13-0

    You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.19.4
    ```
3.  Upgrade k8s to version 1.19.4 - `kubeadm upgrade node v1.19.4`
    ```bash
    root@master-1:~# kubectl version --kubeconfig=/home/hadn/.kube/config
    Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.1", GitCommit:"7879fc12a63337efff607952a323df90cdc7a335", GitTreeState:"clean", BuildDate:"2020-04-08T17:38:50Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
    Server Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.4", GitCommit:"d360454c9bcd1634cf4cc52d1867af5491dc9c5f", GitTreeState:"clean", BuildDate:"2020-11-11T13:09:17Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}
    ```
### Upgrade kubelet and kubectl
## Upgrade Worker Nodes
### Upgrade kubeadm
1.  `apt-mark unhold kubeadm`
2.  `apt-get update && apt-get install -y kubeadm=1.19.4-00`
3.  `apt-mark hold kubeadm`
4.  Verify kubeadm version - `kubeadm version`
### Make unscheduleable of Worker node
1.  `kubectl drain worker-1 --ignore-daemonsets` - **should run on master node terminal session**
    ```bash
    node/worker-1 cordoned
    WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-sccp8, kube-system/kube-proxy-xxh5h
    evicting pod kube-system/calico-kube-controllers-8586758878-4z64l
    evicting pod kube-system/coredns-66bff467f8-8fdp7
    evicting pod kube-system/coredns-66bff467f8-zgxt6
    pod/calico-kube-controllers-8586758878-4z64l evicted
    pod/coredns-66bff467f8-zgxt6 evicted
    pod/coredns-66bff467f8-8fdp7 evicted
    node/worker-1 evicted
    ```
3.  Upgrade k8s to version 1.19.4 - `kubeadm upgrade node`
    ```bash
    [upgrade] Reading configuration from the cluster...
    [upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [preflight] Running pre-flight checks
    [preflight] Skipping prepull. Not a control plane node.
    [upgrade] Skipping phase. Not a control plane node.
    [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
    [upgrade] The configuration for this node was successfully updated!
    [upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
    ```
### Upgrade kubelet and kubectl
1.  apt-mark unhold kubelet kubectl
2.  apt-get install -y kubelet=1.19.4-00 kubectl=1.19.4-00
3.  apt-mark hold kubelet kubectl
4.  systemctl daemon-reload
5.  systemctl restart kubelet
6.  kubectl uncordon worker-1 - Now make the master available for the scheduler. **should run on master node terminal session**
7.  kubectl get nodes
## Verify the status of the cluster
1.  kubectl get nodes
1.  kubectl get pods -A
## Reference link
1.   https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
2.   https://platform9.com/blog/kubernetes-upgrade-the-definitive-guide-to-do-it-yourself/
