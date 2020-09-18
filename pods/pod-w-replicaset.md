# Lab Pod with Replica Set
## Lab Requirements
1.  create 3 nginx pod using image `nginx:alpine`
    1.  have to create 3 pod yamls with difference name of each pod
2.  Using replica set to reduce a complex of create 3 pods and maintaining 3 nginx pods are running at anytime
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: pod-with-replicaset
      labels:
        app: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx  # should map to tamplate.metadata.labels
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - name: nginx
              image: nginx:alpine
    ```
3.  check how many pod running: `kubectl get pods`
    ```bash
    hadn@master-1:~/lab$ kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    pod-with-replicaset-bl942         1/1     Running   0          9m5s
    pod-with-replicaset-jt4nm         1/1     Running   0          9m5s
    pod-with-replicaset-zqllr         1/1     Running   0          9m5s
    ```
4.  Delete pod of replica set: `kubectl delete pod $podname`
5.  check how many pod running: `kubectl get pods`
    ```bash
    hadn@master-1:~/lab$ kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    pod-with-replicaset-2wg76         1/1     Running   0          14s
    pod-with-replicaset-jt4nm         1/1     Running   0          9m5s
    pod-with-replicaset-zqllr         1/1     Running   0          9m5s
    ```
