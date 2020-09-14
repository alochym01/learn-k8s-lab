# Lab pod with namespace
## Lab requirements
1.  Create namespace `alochym` - `kubectl apply -f namespace.yaml`
    ```yaml
    # namespace.yaml
    apiVersion: v1
    kind: Namespace
    metadata:
    name: alochym
    ```
2.  Create a pod in namespace `alochym` - `kubectl apply -f pod-w-namespace.yaml`
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx
        namespace: alochym
        labels:
            environment: production
            app: nginx
    spec:
    containers:
        - name: nginx
        image: nginx:alpine
        ports:
            - containerPort: 80
            name: http
            protocol: TCP
    ```
3.  Using kubectl
    1.  list namespace - `kubectl get namespace| kubectl get ns`
        ```bash
        NAME              STATUS   AGE
        alochym           Active   16s
        default           Active   73d
        ingress-nginx     Active   6d21h
        kube-node-lease   Active   73d
        kube-public       Active   73d
        kube-system       Active   73d
        ```
    2.  list pod in namespace - `kubectl get pods -n alochym`
        ```bash
        NAME    READY   STATUS    RESTARTS   AGE
        nginx   1/1     Running   0          21m
        ```
    3.  -   Interacting with pod
            -   Executing cli inside container using kubectl
                -   `kubectl exec $podname -c $container_name -n $namespace_name -- env`
                -   `kubectl exec $podname -n $namespace_name -- env`
            -   Interactive shell inside pod
                -   `kubectl exec -it $podname -c $container_name -n $namespace_name -- sh`
                -   `kubectl exec -it $podname -n $namespace_name -- sh`
4.  Create limit range object for namespace `alochym` - `kubectl apply -f limitrange-alochym.yaml`
    ```yaml
    # limitrange-alochym.yaml
    # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#limitrangeitem-v1-core
    apiVersion: v1
    kind: LimitRange
    metadata:
        name: alochym-namespace
        namespace: alochym
    spec:
        limits:
        -   max:
                memory: 1Gi
            min:
                memory: 500Mi
            type: Container # can be Pod
    ```
    1.  list all limit range object in namespace `alochym` - `kubectl get limitrange -n alochym`
        ```yaml
        NAME                CREATED AT
        alochym-namespace   2020-09-14T08:44:06Z
        ```
    2.  check pod create in step 2 for resource request/ resource limit - `kubectl get pods nginx -n alochym -o yaml` => there is no resource apply from limit range
    3.  delete pod in step 2 and recreate pod
        1.  check pod create in step 4.3 and get result => the limit range is apply to pod
