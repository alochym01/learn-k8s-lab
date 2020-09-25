# Lab Pod Anti Affinity
## Requirements
1.  create node label `disktype=ssd` - `kubectl label node $node-name disktype=ssd`
2.  Create redis deployment with 3 replica and should be run only 1 pod/node
    ```yaml
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
                    # run pod on a node with label disktype
                    # do not schedule pod if there is a pod with label app=redis which is running in a node
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
    gin-deployment-7b67858ddd-pn8tj   1/1     Running   0          4d9h   172.16.226.103   worker-1   <none>           <none>
    gin-deployment-7b67858ddd-pxc66   1/1     Running   0          4d9h   172.16.226.108   worker-1   <none>           <none>
    gin-deployment-7b67858ddd-r8dlv   1/1     Running   0          4d9h   172.16.226.101   worker-1   <none>           <none>
    gin-deployment-7b67858ddd-rfsbf   1/1     Running   0          4d9h   172.16.226.102   worker-1   <none>           <none>
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
