# Lab Node affinity
## Node Affinity with preferredDuringScheduling
0.  check labels of all nodes - `kubectl get nodes --show-labels=true`
    ```bash
    NAME       STATUS   ROLES    AGE   VERSION   LABELS
    master-1   Ready    master   83d   v1.18.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master-1,kubernetes.io/os=linux,node-role.kubernetes.io/master=
    worker-1   Ready    <none>   16d   v1.18.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker-1,kubernetes.io/os=linux
    ```
1.  create pod yaml
    ```yaml
    # pod-w-schedule-node-affinity-prefer.yaml
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
            preferredDuringSchedulingIgnoredDuringExecution:
            -   weight: 1
                preference:
                    matchExpressions:
                    -   key: disktype
                        operator: In
                        values:
                        - ssd
    containers:
    -   name: nginx
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
    nginx                             1/1     Running   0          7s

    hadn@master-1:~/lab$ kubectl get event
    LAST SEEN   TYPE      REASON             OBJECT                                     MESSAGE
    8s          Normal    Pulled             pod/nginx   Container image "nginx:alpine" already present on machine
    7s          Normal    Created            pod/nginx   Created container nginx
    7s          Normal    Started            pod/nginx   Started container nginx
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
5.  remove label - `kubectl label node worker-1 disktype-`
