# Working with Namespace

## Question
-   Create a Namespace ckad-ns1 in your cluster. In this namespace, run the following Pods:
    1.  A Pod with the name pod-a, running the httpd server image
    2.  A Pod with the name pod-b, running the nginx server image as well as the alpine image

## Solution
1.  Create namespace with --dry-run cli
    1.  `kubectl create ns ckad-ns1 --dry-run=client -o yaml > ns-ckad-ns-1.yaml`
        ```bash
        apiVersion: v1
        kind: Namespace
        metadata:
          creationTimestamp: null
          name: ckad-ns1
        spec: {}
        status: {}
        ```
    2.  `kubectl apply -f ns-ckad-ns1.yaml`
        ```bash
        namespace/ckad-ns1 created
        ```
    3.  `kubectl get ns`
        ```bash
        NAME              STATUS   AGE
        ckad-ns1          Active   6s
        default           Active   125d
        kube-node-lease   Active   125d
        kube-public       Active   125d
        kube-system       Active   125d
        ```
2.  Create A Pod with the name pod-a, running the httpd server image in ckad-ns1 namespace
    1.  `kubectl run pod-a --image=httpd --dry-run=client -o yaml > pod-a.yaml`
    2.  edit pod-a.yaml and then append `namespace: ckad-ns1`
        ```bash
        apiVersion: v1
        kind: Pod
        metadata:
          creationTimestamp: null
          labels:
            run: pod-a
          name: pod-a
          namespace: ckad-ns1
        spec:
          containers:
          - image: httpd
            name: pod-a
            resources: {}
          dnsPolicy: ClusterFirst
          restartPolicy: Always
        status: {}
        ```
    3.  `kubectl apply -f pod-a.yaml`
        ```bash
        pod/pod-a created
        ```
    4.  get ip and test httpd server
        ```bash
        kubectl get pods -n ckad-ns1 -o wide
    
        NAME    READY   STATUS    RESTARTS   AGE   IP              NODE       NOMINATED NODE   READINESS GATES
        pod-a   1/1     Running   0          30s   172.16.226.94   worker-1   <none>           <none>

        curl 172.16.226.94 -vv

        * Rebuilt URL to: 172.16.226.94/
        *   Trying 172.16.226.94...
        * TCP_NODELAY set
        * Connected to 172.16.226.94 (172.16.226.94) port 80 (#0)
        > GET / HTTP/1.1
        > Host: 172.16.226.94
        > User-Agent: curl/7.58.0
        > Accept: */*
        > 
        < HTTP/1.1 200 OK
        < Date: Thu, 05 Nov 2020 08:10:00 GMT
        < Server: Apache/2.4.46 (Unix)
        < Last-Modified: Mon, 11 Jun 2007 18:53:14 GMT
        < ETag: "2d-432a5e4a73a80"
        < Accept-Ranges: bytes
        < Content-Length: 45
        < Content-Type: text/html
        < 
        <html><body><h1>It works!</h1></body></html>
        * Connection #0 to host 172.16.226.94 left intact
        ```
    5.  reference httpd server - https://hub.docker.com/_/httpd

3.  A Pod with the name pod-b, running the nginx server image as well as the alpine image
    1.  `kubectl run pod-b --image=nginx:1.18.0-alpine --dry-run=client -o yaml > pod-b.yaml`
    2.  edit pod-b.yaml and then append `namespace: ckad-ns1`, add image alpine
        ```bash
        apiVersion: v1
        kind: Pod
        metadata:
          creationTimestamp: null
          labels:
            run: pod-b
          namespace: ckad-ns1
          name: pod-b
        spec:
          containers:
          - image: alpine
            name: alpine
            resources: {}
          - image: nginx:1.18.0-alpine
            name: nginx
            resources: {}
          dnsPolicy: ClusterFirst
          restartPolicy: Always
        status: {}
        ```
    3.  `kubectl apply -f pod-b.yaml`
        ```bash
        pod/pod-b created
        ```
    4.  get ip address and test nginx-alpine server
        ```bash
        kubectl get pods -n ckad-ns1 -o wide

        NAME    READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
        pod-a   1/1     Running   0          32m   172.16.226.94    worker-1   <none>           <none>
        pod-b   2/2     Running   0          12s   172.16.226.110   worker-1   <none>           <none>

        curl http://172.16.226.110

        <!DOCTYPE html>
        <html>
        <head>
        <title>Welcome to nginx!</title>
        <style>
            body {
                width: 35em;
                margin: 0 auto;
                font-family: Tahoma, Verdana, Arial, sans-serif;
            }
        </style>
        </head>
        <body>
        <h1>Welcome to nginx!</h1>
        <p>If you see this page, the nginx web server is successfully installed and
        working. Further configuration is required.</p>

        <p>For online documentation and support please refer to
        <a href="http://nginx.org/">nginx.org</a>.<br/>
        Commercial support is available at
        <a href="http://nginx.com/">nginx.com</a>.</p>

        <p><em>Thank you for using nginx.</em></p>
        </body>
        </html>
        ```
    5.  reference link - https://hub.docker.com/_/nginx, https://brandonwillmott.com/2020/09/03/backup-and-restore-etcd-in-kubernetes-cluster-for-cka-v1-19/

