# Lab Pod with deployment with RollingUpdate
## Lab requirements
1.  Create 2 nginx web server
    ```yaml
    # pod-w-deployment-rollingupdate-strategy.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: web-roll-deployment
        labels:
            app: web-deployment
    spec:
        replicas: 2
        strategy:
            type: RollingUpdate
            rollingUpdate:
                maxUnavailable: 20%
                maxSurge: 20%
        minReadySeconds: 10
        progressDeadlineSeconds: 600
        selector:
            matchLabels:
                app: web
        template:
            metadata:
                labels:
                    app: web
            spec:
                containers:
                    -   name: nginx
                        image: nginx:1.18.0-alpine
    ```
2.  Update nginx at step 1 with newer version 1.19.2
    1.  edit `pod-w-deployment-rollingupdate-strategy.yaml` and change image to `1.19.2-alpine`
    2.  kubectl apply -f `pod-w-deployment-rollingupdate-strategy.yaml`
        ```yaml
        NAME                                   READY   STATUS              RESTARTS   AGE
        web-roll-deployment-7bcb84fccc-dcsgq   0/1     ContainerCreating   0          4s
        web-roll-deployment-7db79b4b9d-6lrdz   1/1     Running             0          7m33s
        web-roll-deployment-7db79b4b9d-x7r6b   1/1     Running             0          7m18s

        NAME                                   READY   STATUS              RESTARTS   AGE
        web-roll-deployment-7bcb84fccc-4mbtl   0/1     ContainerCreating   0          3s
        web-roll-deployment-7bcb84fccc-dcsgq   1/1     Running             0          17s
        web-roll-deployment-7db79b4b9d-6lrdz   1/1     Running             0          7m46s
        web-roll-deployment-7db79b4b9d-x7r6b   1/1     Terminating         0          7m31s
        ```
    3.  monitor deployment update `kubectl rollout status deployments web-deployment`
        ```yaml
        Waiting for deployment "web-roll-deployment" rollout to finish: 1 out of 2 new replicas have been updated...
        Waiting for deployment "web-roll-deployment" rollout to finish: 1 out of 2 new replicas have been updated...
        Waiting for deployment "web-roll-deployment" rollout to finish: 1 out of 2 new replicas have been updated...
        Waiting for deployment "web-roll-deployment" rollout to finish: 1 out of 2 new replicas have been updated...
        Waiting for deployment "web-roll-deployment" rollout to finish: 1 old replicas are pending termination...
        Waiting for deployment "web-roll-deployment" rollout to finish: 1 old replicas are pending termination...
        Waiting for deployment "web-roll-deployment" rollout to finish: 1 old replicas are pending termination...
        deployment "web-roll-deployment" successfully rolled out
        ```
    4.  check history of rollout - `kubectl rollout history deployment/web-roll-deployment`
        ```yaml
        deployment.apps/web-roll-deployment
        REVISION  CHANGE-CAUSE
        1         <none>
        2         <none>
        ```
3.  Scale up to 5 nginx web server - `kubectl scale deployment web-roll-deployment –replicas=5` ==> option, prefer is editing yaml file
4.  Scale down to 2 nginx web server - `kubectl scale deployment web-roll-deployment –replicas=2` ==> option, prefer is editing yaml file
5.  Rollback to original version
    1.  edit `pod-w-deployment-rollingupdate-strategy.yaml` and change image to `1.18.0-alpine`
    2.  check history of rollout - `kubectl rollout history deployment/web-roll-deployment`
        ```yaml
        deployment.apps/web-roll-deployment
        REVISION  CHANGE-CAUSE
        2         <none>
        3         <none>
        ```
    3.  kubectl apply -f `pod-w-deployment-rollingupdate-strategy.yaml | kubectl rollout undo deployments kuard --to-revision=revision_No`
    4.  monitor deployment update `kubectl rollout status deployments/web-roll-deployment`
