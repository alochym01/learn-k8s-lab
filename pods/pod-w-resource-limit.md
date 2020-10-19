# Lab Pods with label yaml
## Lab requirements
-   Create a nginx POD that is running 1.18-0 alpine image.
-   The POD is created with minium resource
    -   cpu=500m(1/2 cpu)
    -   memory=128Mi
1.  Create a simple pod file using `nginx alpine` with format yaml
    ```yaml
    # simple-alpine-w-label-nginx.yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: pod-with-resource-request
    spec:
        containers:
        -   name: nginx
            image: nginx:1.18.0-alpine
            resources:
                requests:
                cpu: "500m"
                memory: "128Mi"
            ports:
            -   containerPort: 80
                name: http
                protocol: TCP
    ```
2.  Using kubectl to:
    -   apply pod to k8s - `kubectl apply -f simple-alpine-w-label-nginx.yaml`
    -   Access to pod - `kubectl exec pod-with-resource-request -it -- sh`
        -   using `top` cli to see all cpu and memory of pod
            ```bash
            Mem: 2821684K used, 5238532K free, 3004K shrd, 728204K buff, 1324904K cached
            CPU0:   0% usr   0% sys   0% nic 100% idle   0% io   0% irq   0% sirq
            CPU1:   0% usr   0% sys   0% nic 100% idle   0% io   0% irq   0% sirq
            CPU2:   0% usr   0% sys   0% nic 100% idle   0% io   0% irq   0% sirq
            CPU3:   0% usr  16% sys   0% nic  83% idle   0% io   0% irq   0% sirq
            Load average: 0.18 0.24 0.25 3/784 55
            PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
            30     1 nginx    S     6424   0%   3   0% nginx: worker process
            29     1 nginx    S     6424   0%   2   0% nginx: worker process
            32     1 nginx    S     6424   0%   1   0% nginx: worker process
            31     1 nginx    S     6424   0%   0   0% nginx: worker process
             1     0 root     S     5984   0%   1   0% nginx: master process nginx -g daemon off;
            49     0 root     S     1648   0%   1   0% sh
            33     0 root     S     1580   0%   2   0% top
            55    49 root     R     1580   0%   3   0% top
            ```
    -   get event after apply pods - `kubectl get events | grep -i $podname`
    ```bash
    15s         Normal    Scheduled   pod/pod-with-resource-request   Successfully assigned default/pod-with-resource-request to worker-1
    13s         Normal    Pulled      pod/pod-with-resource-request   Container image "nginx:1.18.0-alpine" already present on machine
    12s         Normal    Created     pod/pod-with-resource-request   Created container nginx
    11s         Normal    Started     pod/pod-with-resource-request   Started container nginx
    ```
    -   get all pod output with labels - `kubectl get pods $podname --show-labels -o [wide|yaml|json]`
    ```bash
    NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES   LABELS
    nginx   1/1     Running   0          14m   172.17.0.3   minikube   <none>           <none>            app=nginx,environment=production
    ```
    -   get all pod with labels
        -   `kubectl get pods --selector key=value -o [wide|yaml|json]`
        -   `kubectl get pods -l key=value -o [wide|yaml|json]`
    ```bash
    NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
    nginx   1/1     Running   0          36m   172.17.0.3   minikube   <none>           <none>
    ```
    -   get details of pod - `kubectl describe pods $podname`
    ```bash
    Name:         nginx
    Namespace:    default
    Priority:     0
    Node:         minikube/192.168.99.100
    Start Time:   Sat, 12 Sep 2020 16:52:27 +0700
    Labels:       app=nginx
                  environment=production
    Annotations:  <none>
    Status:       Running
    IP:           172.17.0.3
    IPs:
    IP:  172.17.0.3
    Containers:
    nginx:
        Container ID:   docker://b5551bbc2f7f2c9d675641d67f6ea088630e52e44c1ba762095174aa99690772
        Image:          nginx:alpine
        Image ID:       docker-pullable://nginx@sha256:a97eb9ecc708c8aa715ccfb5e9338f5456e4b65575daf304f108301f3b497314
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
        Started:        Sat, 12 Sep 2020 16:52:43 +0700
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:         /var/run/secrets/kubernetes.io/serviceaccount from default-token-wltt4 (ro)
    Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
    Volumes:
    default-token-wltt4:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-wltt4
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                     node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
    Type     Reason       Age   From               Message
    ----     ------       ----  ----               -------
    Normal   Scheduled    39m                      Successfully assigned default/nginx to minikube
    Normal   Pulling      39m   kubelet, minikube  Pulling image "nginx:alpine"
    Normal   Pulled       39m   kubelet, minikube  Successfully pulled image "nginx:alpine" in 13.860887637s
    Normal   Created      39m   kubelet, minikube  Created container nginx
    Normal   Started      39m   kubelet, minikube  Started container nginx
    ```
    -   Interacting with pod
        -   Executing cli inside container using kubectl
            -   `kubectl exec $podname -c $container_name -- env`
            -   `kubectl exec $podname -- env`
        -   Interactive shell inside pod
            -   `kubectl exec -it $podname -c $container_name -- sh`
            -   `kubectl exec -it $podname -- sh`
    -   Get logs of pod
        -   Print the logs for a pod - `kubectl logs $podname | kubectl logs -f $podname`
        -   Print the logs for the last hour for a pod - `kubectl logs --since=1h $podname`
        -   Get the most recent 20 lines of logs - `kubectl logs --tail=20 $podname`
        -   Get logs from a service and optionally select which container
        -   if there are multiple containers inside pod, you should use `kubectl logs -c $container_name $podname`
        -   get the logs for a previously failed pod - `kubectl logs --previous $podname`
    -   Explain each section of pod
        -   `kubectl explain pods`
        -   `kubectl explain pods.apiVersion`
        -   `kubectl explain pods.kind`
        -   `kubectl explain pods.metadata`
        -   `kubectl explain pods.spec`
        -   `kubectl explain pods.status`
    -   Show metrics of pods - require metric server - `kubectl top pod $podname`
        -   `minikube addons enable metrics-server`
