# Lab Pod Affinity and Anti Affinity
## Requirements
1.  Create node label `disktype=ssd` - `kubectl label node $node-name disktype=ssd`
2.  Create redis deployment with 3 replica and should be run only 1 pod/node
    ```yaml
    # podAntiAffinity
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: redis
    spec:
        selector:
            matchLabels:
                app: redis
        replicas: 3
        template:
            metadata:
                labels:
                    app: redis
            spec:
                affinity:
                    podAntiAffinity:
                        requiredDuringSchedulingIgnoredDuringExecution:
                        -   labelSelector:
                                matchExpressions:
                                -   key: app
                                    operator: In
                                    values:
                                    - redis
                            topologyKey: "disktype"
                containers:
                    -   name: redis-server
                        image: redis:3.2-alpine
    ```
3.  Check result - `kubectl get pods -owide`
    ```bash
    NAME                              READY   STATUS    RESTARTS   AGE    IP               NODE       NOMINATED NODE   READINESS GATES
    redis-5684c46645-8nfjp            1/1     Running   0          94s    172.16.226.72    worker-1   <none>           <none>
    redis-5684c46645-n8xxc            0/1     Pending   0          94s    <none>           <none>     <none>           <none>
    redis-5684c46645-wrtpj            0/1     Pending   0          94s    <none>           <none>     <none>           <none>
    ```
4.  Check event - `kubectl get  event`
    ```bash
    LAST SEEN   TYPE      REASON              OBJECT                        MESSAGE
    2m19s       Normal    Scheduled           pod/redis-5684c46645-8nfjp    Successfully assigned default/redis-5684c46645-8nfjp to worker-1
    2m16s       Normal    Pulling             pod/redis-5684c46645-8nfjp    Pulling image "redis:3.2-alpine"
    2m8s        Normal    Pulled              pod/redis-5684c46645-8nfjp    Successfully pulled image "redis:3.2-alpine"
    2m7s        Normal    Created             pod/redis-5684c46645-8nfjp    Created container redis-server
    2m6s        Normal    Started             pod/redis-5684c46645-8nfjp    Started container redis-server
    57s         Warning   FailedScheduling    pod/redis-5684c46645-n8xxc    0/2 nodes are available: 1 node(s) didn't match pod affinity/anti-affinity, 1 node(s) didn't satisfy existing pods anti-affinity rules, 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
    57s         Warning   FailedScheduling    pod/redis-5684c46645-wrtpj    0/2 nodes are available: 1 node(s) didn't match pod affinity/anti-affinity, 1 node(s) didn't satisfy existing pods anti-affinity rules, 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
    2m19s       Normal    SuccessfulCreate    replicaset/redis-5684c46645   Created pod: redis-5684c46645-8nfjp
    2m19s       Normal    SuccessfulCreate    replicaset/redis-5684c46645   Created pod: redis-5684c46645-n8xxc
    2m19s       Normal    SuccessfulCreate    replicaset/redis-5684c46645   Created pod: redis-5684c46645-wrtpj
    2m19s       Normal    ScalingReplicaSet   deployment/redis              Scaled up replica set redis-5684c46645 to 3
    ```
5.  Create web deployment with 3 replica
    1.  should be run only 1 pod/node
    2.  only run on a node if there is a pod with label `app=redis` that already run
    ```yaml
    # define podAffinity and podAntiAffinity with deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: web
    spec:
        selector:
            matchLabels:
                app: web
        replicas: 3
        template:
            metadata:
            labels:
                app: web
            spec:
                affinity:
                    # run pod on a node with label disktype
                    # do not schedule pod if there is a pod with label app=web which is running in a node
                    podAntiAffinity:
                        requiredDuringSchedulingIgnoredDuringExecution:
                        -   labelSelector:
                                matchExpressions:
                                -   key: app
                                    operator: In
                                    values:
                                    - web
                            topologyKey: "disktype"
                    podAffinity:
                    # run pod on a node with label disktype
                    # and try to schedule a pod if there is a pod with label app=redis which is running in a node
                        requiredDuringSchedulingIgnoredDuringExecution:
                        -   labelSelector:
                                matchExpressions:
                                -   key: app
                                    operator: In
                                    values:
                                    - redis
                            topologyKey: "disktype"
            containers:
                -   name: web-app
                    image: nginx:1.18.0-alpine
    ```
6.  Check result - `kubectl get pods -owide`
    ```yaml
    NAME                              READY   STATUS    RESTARTS   AGE     IP               NODE       NOMINATED NODE   READINESS GATES
    redis-5684c46645-8nfjp            1/1     Running   0          50m     172.16.226.72    worker-1   <none>           <none>
    redis-5684c46645-n8xxc            0/1     Pending   0          50m     <none>           <none>     <none>           <none>
    redis-5684c46645-wrtpj            0/1     Pending   0          50m     <none>           <none>     <none>           <none>
    web-84fb7674d4-m49z9              0/1     Pending   0          4s      <none>           <none>     <none>           <none>
    web-84fb7674d4-sj4qr              0/1     Pending   0          4s      <none>           <none>     <none>           <none>
    web-84fb7674d4-zwnzn              1/1     Running   0          4s      172.16.226.68    worker-1   <none>           <none>
    ```
7.  Check event - `kubectl get  event`
    ```yaml
    LAST SEEN   TYPE      REASON              OBJECT                        MESSAGE
    2m57s       Warning   FailedScheduling    pod/web-b8c54896d-hm294       0/2 nodes are available: 1 node(s) didn't match pod affinity rules, 1 node(s) didn't match pod affinity/anti-affinity, 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
    89s         Warning   FailedScheduling    pod/web-b8c54896d-hm294       skip schedule deleting pod: default/web-b8c54896d-hm294
    2m57s       Warning   FailedScheduling    pod/web-b8c54896d-ts4c8       0/2 nodes are available: 1 node(s) didn't match pod affinity rules, 1 node(s) didn't match pod affinity/anti-affinity, 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
    2m6s        Warning   FailedScheduling    pod/web-b8c54896d-ts4c8       skip schedule deleting pod: default/web-b8c54896d-ts4c8
    2m57s       Warning   FailedScheduling    pod/web-b8c54896d-vrdkj       0/2 nodes are available: 1 node(s) didn't match pod affinity rules, 1 node(s) didn't match pod affinity/anti-affinity, 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
    89s         Warning   FailedScheduling    pod/web-b8c54896d-vrdkj       skip schedule deleting pod: default/web-b8c54896d-vrdkj

    ```
