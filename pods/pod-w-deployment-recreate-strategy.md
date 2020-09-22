# Lab Pod with deployment with Recreate Strategy
## Lab requirements
1.  Create 2 nginx web server
    ```yaml
    # pod-w-deployment-recreate-strategy.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: web-deployment
        labels:
            app: web-deployment
    spec:
        replicas: 2
        strategy:
            type: Recreate
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
    1.  edit `pod-w-deployment-recreate-strategy.yaml` and change image to `1.19.2-alpine`
    2.  kubectl apply -f `pod-w-deployment-recreate-strategy.yaml`
        ```yaml
        NAME                              READY   STATUS        RESTARTS   AGE
        web-deployment-7bcb84fccc-7fwlx   1/1     Terminating   0          2m26s
        web-deployment-7bcb84fccc-kgpx7   1/1     Terminating   0          2m26s

        NAME                              READY   STATUS        RESTARTS   AGE
        web-deployment-7db79b4b9d-hhnb4   0/1     ContainerCreating   0          3s
        web-deployment-7db79b4b9d-s6fkk   0/1     ContainerCreating   0          3s

        NAME                              READY   STATUS        RESTARTS   AGE
        web-deployment-7db79b4b9d-hhnb4   1/1     Running   0          58s
        web-deployment-7db79b4b9d-s6fkk   1/1     Running   0          58s
        ```
    3.  monitor deployment update `kubectl rollout status deployments/web-deployment($deployment_name)`
        ```yaml
        Waiting for deployment "web-deployment" rollout to finish: 0 out of 2 new replicas have been updated...
        Waiting for deployment "web-deployment" rollout to finish: 0 out of 2 new replicas have been updated...
        Waiting for deployment "web-deployment" rollout to finish: 0 of 2 updated replicas are available...
        Waiting for deployment "web-deployment" rollout to finish: 1 of 2 updated replicas are available...
        deployment "web-deployment" successfully rolled out
        ```
    4.  check history of rollout - `kubectl rollout history deployment/web-deployment`
        ```bash
        REVISION  CHANGE-CAUSE
        1         <none>
        2         <none>
        ```
3.  Scale up to 5 nginx web server - `kubectl scale deployment web-deployment –replicas=5` ==> option, prefer is editing yaml file
4.  Scale down to 2 nginx web server - `kubectl scale deployment web-deployment –replicas=2` ==> option, prefer is editing yaml file
5.  Rollback to original version
    1.  edit `pod-w-deployment-recreate-strategy.yaml` and change image to `1.18.0-alpine`
    2.  `kubectl apply -f pod-w-deployment-recreate-strategy.yaml | kubectl rollout undo deployments web-deployment --to-revision=revision_No`
    3.  monitor deployment update `kubectl rollout status deployments/web-deployment`
    4.  check history of rollout - `kubectl rollout history deployment/web-deployment`
        ```bash
        REVISION  CHANGE-CAUSE
        2         <none>
        3         <none>
        ```
