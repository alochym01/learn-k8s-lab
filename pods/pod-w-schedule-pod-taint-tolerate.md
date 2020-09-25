# Lab pod with taint and tolerate
## Requirement
0.  Taint a node with taint `alochym=ssd` - `kubectl taint nodes worker-1 alochym=ssd:NoSchedule`
1.  Create a normal pod
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx-tolerate
        labels:
            env: test
    spec:
        containers:
        -   name: nginx
            image: nginx:1.18.0-alpine
    ```
2.  Check result
    1.  `kubectl get event`
        ```bash
        LAST SEEN   TYPE      REASON              OBJECT                       MESSAGE
        18s         Warning   FailedScheduling    pod/nginx-tolerate           0/2 nodes are available: 1 node(s) had taint {alochym: ssd}, that the pod didn't tolerate, 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
        ```
    2.  `kubectl get pods`
        ```bash
        NAME                              READY   STATUS    RESTARTS   AGE     IP               NODE       NOMINATED NODE   READINESS GATES
        nginx-tolerate                    0/1     Pending   0          5m13s   <none>           <none>     <none>           <none>
        ```
3.  Create a pod with tolerate `alochym=ssd`
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx-ok
        labels:
            env: test
    spec:
        containers:
        -   name: nginx
            image: nginx:1.18.0-alpine
        tolerations:
        -   key: "alochym"
            # the operator is Equal and the values are equal.
            # the operator is Exists (in which case no value should be specified)
            operator: "Exists"
            # NoExecute, PreferNoSchedule, NoSchedule
            effect: "NoSchedule"
    ```
4.  Check result - `kubectl get pods`
    1.  `kubectl get event`
        ```bash
        LAST SEEN   TYPE      REASON              OBJECT                       MESSAGE
        16s         Normal    Scheduled           pod/nginx-ok                 Successfully assigned default/nginx-ok to worker-1
        13s         Normal    Pulled              pod/nginx-ok                 Container image "nginx:1.18.0-alpine" already present on machine
        12s         Normal    Created             pod/nginx-ok                 Created container nginx
        12s         Normal    Started             pod/nginx-ok                 Started container nginx
        ```
    2.  `kubectl get pods`
        ```bash
        NAME                              READY   STATUS    RESTARTS   AGE     IP               NODE       NOMINATED NODE   READINESS GATES
        nginx-ok                          1/1     Running   0          10s     172.16.226.66    worker-1   <none>           <none>
        nginx-tolerate                    0/1     Pending   0          5m13s   <none>           <none>     <none>           <none>
        ```
5.  remove taint - `kubectl taint nodes worker-1 alochym:NoSchedule-`
