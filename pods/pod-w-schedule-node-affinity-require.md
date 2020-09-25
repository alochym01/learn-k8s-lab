# Lab Node affinity
## Node Affinity with requireDuringScheduling
0.  check labels of all nodes - `kubectl get nodes --show-labels=true`
    ```bash
    NAME       STATUS   ROLES    AGE   VERSION   LABELS
    master-1   Ready    master   83d   v1.18.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master-1,kubernetes.io/os=linux,node-role.kubernetes.io/master=
    worker-1   Ready    <none>   16d   v1.18.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker-1,kubernetes.io/os=linux
    ```
1.  create pod yaml
    ```yaml
    # pod-w-schedule-node-affinity-require.yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx
        labels:
            environment: production
            app: nginx
    spec:
        affinity:
            nodeAffinity:
                requiredDuringSchedulingIgnoredDuringExecution:
                    nodeSelectorTerms:
                    -   matchExpressions:
                        -   key: disktype
                            operator: In
                            values:
                            -   ssd
        containers:
        - name: nginx
            image: nginx:alpine
            ports:
            -   containerPort: 80
                name: http
                protocol: TCP
    ```
2.  check pod is scheduled
    ```bash
    hadn@master-1:~/lab$ kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    nginx                             0/1     Pending   0          111s

    hadn@master-1:~/lab$ kubectl get event
    LAST SEEN   TYPE      REASON             OBJECT                                     MESSAGE
    30s         Warning   FailedScheduling   pod/nginx                                  0/2 nodes are available: 2 node(s) didn't match node selector.
    ```
3.  create label `disktype=ssd` for worker-1
    ```bash
    hadn@master-1:~/lab$ kubectl label node worker-1 disktype=ssd
    node/worker-1 labeled
    ```
4.  check label of worker-1 - `kubectl get nodes -l  disktype=ssd`
    ```bash
    NAME       STATUS   ROLES    AGE   VERSION   LABELS
    master-1   Ready    master   83d   v1.18.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master-1,kubernetes.io/os=linux,node-role.kubernetes.io/master=
    worker-1   Ready    <none>   16d   v1.18.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker-1,kubernetes.io/os=linux
    ```
5.  check pod is scheduled
    ```bash
    hadn@master-1:~/lab$ kubectl get event
    LAST SEEN   TYPE      REASON             OBJECT                                     MESSAGE
    30s         Warning   FailedScheduling   pod/nginx                                  0/2 nodes are available: 2 node(s) didn't match node selector.
    10s         Normal    Scheduled          pod/nginx                                  Successfully assigned default/nginx to worker-1
    8s          Normal    Pulled             pod/nginx                                  Container image "nginx:alpine" already present on machine

    hadn@master-1:~/lab$ kubectl get pods
    NAME                              READY   STATUS              RESTARTS   AGE
    nginx                             0/1     ContainerCreating   0          3m12s

    hadn@master-1:~/lab$ kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    nginx                             1/1     Running   0          3m16s
    ```
6.  remove label - `kubectl label node worker-1 disktype-`
