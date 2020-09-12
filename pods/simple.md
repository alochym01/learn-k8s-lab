# Lab Pods with simple yaml
## Lab requirements

1.  Create a simple pod file using `nginx alpine` with format yaml
```yaml
# simple-alpine-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
      name: http
      protocol: TCP
```
2.  Using kubectl to:
    -   apply pod to k8s - `kubectl apply -f simple-alpine-nginx.yaml`
    -   get event after apply pods - `kubectl get events | grep -i $podname`
    ```bash
    16m         Normal    Scheduled     pod/nginx   Successfully assigned default/nginx to minikube
    16m         Warning   FailedMount   pod/nginx   MountVolume.SetUp failed for volume "default-token-wltt4" : failed to sync secret cache: timed out waiting for the condition
    16m         Normal    Pulling       pod/nginx   Pulling image "nginx:alpine"
    16m         Normal    Pulled        pod/nginx   Successfully pulled image "nginx:alpine" in 13.860887637s
    16m         Normal    Created       pod/nginx   Created container nginx
    16m         Normal    Started       pod/nginx   Started container nginx
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
    Node:         minikube/192.168.99.100
    Start Time:   Sat, 12 Sep 2020 16:52:27 +0700
    Labels:       <none>
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
        Started:      Sat, 12 Sep 2020 16:52:43 +0700
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from default-token-wltt4 (ro)
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
    Warning  FailedMount  39m   kubelet, minikube  MountVolume.SetUp failed for volume "default-token-wltt4" : failed to sync secret cache: timed out waiting for the condition
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