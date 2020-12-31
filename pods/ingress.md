# K8s using ingress
1.  Step
    1.  create deployment
    2.  create service expose deployment
    3.  create ingress-resource
    4.  check ingress controller ip address - nginx ingress controller
2.  Create nginx deployment
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        creationTimestamp: null
        labels:
            app: google-web
        name: google-web
        namespace: cka
    spec:
        replicas: 5
        selector:
            matchLabels:
                app: google-web
        template:
            metadata:
                creationTimestamp: null
                labels:
                    app: google-web
            spec:
                containers:
                -   image: gcr.io/google-samples/hello-app:1.0
                    name: hello-app
                    ports:
                    -   name: google-web
                        containerPort: 8080
                        protocol: TCP
                    resources: {}
    status: {}
    ```
3.  Create service expose deployment
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
        creationTimestamp: null
        labels:
            app: google-web
        name: google-web
        namespace: cka
    spec:
        ports:
        -   port: 80
            protocol: TCP
            targetPort: google-web
        selector:
            app: google-web
    status:
    loadBalancer: {}
    ```
    1.  Check service
        ```bash
        hadn@worker-1:~/nginx-ingress$ kubectl describe  svc -n cka google-web
        Name:              google-web
        Namespace:         cka
        Labels:            app=google-web
        Annotations:       <none>
        Selector:          app=google-web
        Type:              ClusterIP
        IP:                10.108.153.77
        Port:              <unset>  80/TCP
        TargetPort:        8080/TCP
        Endpoints:         172.16.226.80:8080,172.16.226.81:8080,172.16.226.82:8080 + 2 more...
        Session Affinity:  None
        Events:            <none>
        ```
4.  Create ingress resource
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
        name: example-ingress
        namespace: cka
        annotations:
            nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
        rules:
        -   host: alochym.com
            http:
                paths:
                -   path: /hello
                    pathType: Prefix
                    backend:
                        service:
                            name: google-web
                            port:
                                number: 80
    ```
    1.  Check Ingress resource
        ```bash
        hadn@worker-1:~/nginx-ingress$ kubectl describe ingress -n cka  example-ingress
        Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
        Name:             example-ingress
        Namespace:        cka
        Address:          192.168.2.20
        Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
        Rules:
            Host         Path  Backends
            ----         ----  --------
            alochym.com
                         /hello   google-web:80   172.16.226.80:8080,172.16.226.81:8080,172.16.226.82:8080 + 2 more...)
        Annotations:   nginx.ingress.kubernetes.io/rewrite-target: /
        Events:        <none>
        ```
5.  Check Ingress controller Service IP Address
    ```bash
    hadn@worker-1:~/nginx-ingress$ kubectl get svc -n ingress-nginx
    NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    ingress-nginx-controller             NodePort    10.96.9.135      <none>        80:32430/TCP,443:31825/TCP   27h
    ingress-nginx-controller-admission   ClusterIP   10.103.181.190   <none>        443/TCP                      27h
    ```
6.  Test using curl
    ```bash
    hadn@worker-1:~/nginx-ingress$ kubectl get nodes -owide
    NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP
    master-1   Ready    master   28h   v1.19.0   192.168.2.150
    worker-1   Ready    <none>   28h   v1.19.0   192.168.2.20

    hadn@worker-1:~/nginx-ingress$ curl --header "Host: alochym.com" 192.168.2.20:32430/hello
    Hello, world!
    Version: 1.0.0
    Hostname: google-web-7d89d855fc-nrgcr

    hadn@worker-1:~/nginx-ingress$ curl --header "Host: alochym.com" 192.168.2.150:32430/hello
    Hello, world!
    Version: 1.0.0
    Hostname: google-web-7d89d855fc-dj4rq
    ```
