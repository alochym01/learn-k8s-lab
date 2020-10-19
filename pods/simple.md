# Lab Pods with simple yaml
## Lab requirements
-   Create a nginx POD that is running 1.18-0 alpine image.

1.  Create a simple pod file using `nginx alpine` with format yaml
    ```yaml
    # simple-alpine-nginx.yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx
    spec:
        containers:
        -   name: nginx
            image: nginx:1.18.0-alpine
            ports:
            -   containerPort: 80
                name: http
                protocol: TCP
    ```
2.  Using kubectl to:
    -   apply pod to k8s - `kubectl apply -f simple-alpine-nginx.yaml`
    -   get event after apply pods - `kubectl get events | grep -i $podname`
        ```bash
        35s         Normal   Scheduled   pod/nginx      Successfully assigned default/nginx to worker-1
        32s         Normal   Pulled      pod/nginx      Container image "nginx:1.18.0-alpine" already present on machine
        31s         Normal   Created     pod/nginx      Created container nginx
        31s         Normal   Started     pod/nginx      Started container nginx
        ```
    -   get all pod output - `kubectl get pods $podname -o [wide|yaml|json]`
        ```bash
        NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
        nginx   1/1     Running   0          36m   172.17.0.3   minikube   <none>           <none>
        ```
    -   get details of pod - `kubectl describe pods $podname`
        ```bash
        Name:         nginx
        Namespace:    default
        Priority:     0
        Node:         worker-1/192.168.2.20
        Start Time:   Mon, 19 Oct 2020 14:59:14 +0700
        Labels:       <none>
        Annotations:  cni.projectcalico.org/podIP: 172.16.226.104/32
                      cni.projectcalico.org/podIPs: 172.16.226.104/32
        Status:       Running
        IP:           172.16.226.104
        IPs:
        IP:  172.16.226.104
        Containers:
        nginx:
            Container ID:   docker://8aa758f08be399a9db0474160bd66a19596d343a5277d90152f5fb9c0ccce201
            Image:          nginx:1.18.0-alpine
            Image ID:       docker-pullable://nginx@sha256:29dc24ed982665eb88598e0129e4ec88c2049fafc63125a4a640dd67529dc6d4
            Port:           80/TCP
            Host Port:      0/TCP
            State:          Running
            Started:      Mon, 19 Oct 2020 14:59:18 +0700
            Ready:          True
            Restart Count:  0
            Environment:    <none>
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from default-token-d8wd9 (ro)
        Conditions:
        Type              Status
        Initialized       True
        Ready             True
        ContainersReady   True
        PodScheduled      True
        Volumes:
        default-token-d8wd9:
            Type:        Secret (a volume populated by a Secret)
            SecretName:  default-token-d8wd9
            Optional:    false
        QoS Class:       BestEffort
        Node-Selectors:  <none>
        Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                         node.kubernetes.io/unreachable:NoExecute for 300s
        Events:
        Type    Reason     Age   From               Message
        ----    ------     ----  ----               -------
        Normal  Scheduled  104s  default-scheduler  Successfully assigned default/nginx to worker-1
        Normal  Pulled     101s  kubelet, worker-1  Container image "nginx:1.18.0-alpine" already present on machine
        Normal  Created    100s  kubelet, worker-1  Created container nginx
        Normal  Started    100s  kubelet, worker-1  Started container nginx
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
