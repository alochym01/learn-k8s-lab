# Lab Pod with liveness
## Requirements
1. create a pod
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-with-http
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          # defines the health checking
          livenessProbe:
            # an http probe
            httpGet:
              path: /_status/healthz
              port: 80
            initialDelaySeconds: 30
            timeoutSeconds: 1
            failureThreshold: 3
            periodSeconds: 10
          ports:
            - containerPort: 80
    ```
2. get health check from pod of container
    1.  container is created after 5 seconds
    2.  the probes must response in 1 second
    3.  HTTP status code > 200 and < 400 will be consider successfull
    4.  if more than 3 failed probes, container will fail and restart
    5.  interval time to probes is 10 seconds
3. check result
   1. using `kubectl get pods` => CrashLoopBackOff - because there is no route for `/_status/healthz`
      ```bash
        NAME                              READY   STATUS             RESTARTS   AGE
        gin-deployment-7b67858ddd-64b29   1/1     Running            0          8h
        gin-deployment-7b67858ddd-dvzvw   1/1     Running            0          8h
        gin-deployment-7b67858ddd-j4xvl   1/1     Running            0          8h
        gin-deployment-7b67858ddd-ldl9s   1/1     Running            0          8h
        pod-with-http                     0/1     CrashLoopBackOff   6          10m
      ```
   2. using `kubectl describe pod pod-w-http`
      ```bash
        Events:
        Type     Reason     Age                     From               Message
        ----     ------     ----                    ----               -------
        Normal   Scheduled  <unknown>               default-scheduler  Successfully assigned default/pod-with-http to worker-1
        Normal   Pulled     9m28s (x4 over 12m)     kubelet, worker-1  Container image "nginx:alpine" already present on machine
        Normal   Killing    9m28s (x3 over 11m)     kubelet, worker-1  Container nginx failed liveness probe, will be restarted
        Normal   Created    9m27s (x4 over 12m)     kubelet, worker-1  Created container nginx
        Normal   Started    9m26s (x4 over 12m)     kubelet, worker-1  Started container nginx
        Warning  Unhealthy  6m48s (x16 over 11m)    kubelet, worker-1  Liveness probe failed: HTTP probe failed with statuscode: 404
        Warning  BackOff    2m16s (x17 over 6m28s)  kubelet, worker-1  Back-off restarting failed container
      ```
