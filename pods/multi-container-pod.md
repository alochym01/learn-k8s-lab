# Multi container
1.  Running 4 nginx container in 1 POD
    1.  kubectl run multi-nginx --image=nginx:1.18-alpine --port=80 --dry-run=client -oyaml > multi-container-pod.yaml
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
            creationTimestamp: null
            labels:
                run: nginx
            name: nginx
        spec:
            containers:
            - image: nginx:1.18-alpine
              name: nginx
              ports:
              - containerPort: 80
              resources: {}
            dnsPolicy: ClusterFirst
            restartPolicy: Always
        status: {}
        ```
    2.  create nginx config file
        ```bash
        server {
            listen       81;
            listen  [::]:81;
            server_name  localhost;

            #charset koi8-r;
            #access_log  /var/log/nginx/host.access.log  main;

            location / {
                root   /usr/share/nginx/html;
                index  index.html index.htm;
            }

            #error_page  404              /404.html;

            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   /usr/share/nginx/html;
            }
        }
        ```
    3.  create 3 configMap for nginx config
        1.  kubectl create configmap nginx-81 --dry-run=client -oyaml --from-file=default-81.conf > nginx-config-81.yaml
            ```yaml
            apiVersion: v1
            data:
                default.conf: |
                    server {
                        listen       81;
                        listen  [::]:81;
                        server_name  localhost;

                        location / {
                            root   /usr/share/nginx/html;
                            index  index.html index.htm;
                        }

                        #error_page  404              /404.html;

                        # redirect server error pages to the static page /50x.html
                        #
                        error_page   500 502 503 504  /50x.html;
                        location = /50x.html {
                            root   /usr/share/nginx/html;
                        }
                    }
            kind: ConfigMap
            metadata:
                creationTimestamp: null
                name: nginx-81
            ```
        2.  kubectl create configmap nginx-82 --dry-run=client -oyaml --from-file=default-82.conf > nginx-config-82.yaml
            ```yaml
            apiVersion: v1
            data:
                default.conf: |
                    server {
                        listen       82;
                        listen  [::]:82;
                        server_name  localhost;

                        location / {
                            root   /usr/share/nginx/html;
                            index  index.html index.htm;
                        }

                        #error_page  404              /404.html;

                        # redirect server error pages to the static page /50x.html
                        #
                        error_page   500 502 503 504  /50x.html;
                        location = /50x.html {
                            root   /usr/share/nginx/html;
                        }
                    }
            kind: ConfigMap
            metadata:
                creationTimestamp: null
                name: nginx-82
            ```
        3.  kubectl create configmap nginx-81 --dry-run=client -oyaml --from-file=default-83.conf > nginx-config-83.yaml
            ```yaml
            apiVersion: v1
            data:
                default.conf: |
                    server {
                        listen       83;
                        listen  [::]:83;
                        server_name  localhost;

                        location / {
                            root   /usr/share/nginx/html;
                            index  index.html index.htm;
                        }

                        #error_page  404              /404.html;

                        # redirect server error pages to the static page /50x.html
                        #
                        error_page   500 502 503 504  /50x.html;
                        location = /50x.html {
                            root   /usr/share/nginx/html;
                        }
                    }
            kind: ConfigMap
            metadata:
                creationTimestamp: null
                name: nginx-83
            ```
        4.  kubectl apply -f nginx-config-81.yaml
        5.  kubectl apply -f nginx-config-82.yaml
        6.  kubectl apply -f nginx-config-83.yaml
    4.  edit POD in step 1 to use configMap as Volume
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
            creationTimestamp: null
            labels:
                run: nginx
            name: nginx
        spec:
            volumes:
            - name: nginx-81
              configMap:
                name: nginx-81
            - name: nginx-82
              configMap:
                name: nginx-82
            - name: nginx-83
              configMap:
                name: nginx-83
            containers:
            - image: nginx:1.18-alpine
              name: nginx
              ports:
              - containerPort: 80
            - image: nginx:1.18-alpine
              name: nginx-81
              ports:
              - containerPort: 81
              volumeMounts:
              - name: nginx-81
                mountPath: /etc/nginx/conf.d
            - image: nginx:1.18-alpine
              name: nginx-82
              ports:
              - containerPort: 82
              volumeMounts:
              - name: nginx-82
                mountPath: /etc/nginx/conf.d
            - image: nginx:1.18-alpine
              name: nginx-83
              ports:
              - containerPort: 83
              volumeMounts:
              - name: nginx-83
                mountPath: /etc/nginx/conf.d
              resources: {}
            dnsPolicy: ClusterFirst
            restartPolicy: Always
        status: {}
        ```
    5.  kubectl apply -f multi-container-pod.yaml
2.  Check a result
    1.  kubectl get pod
        ```bash
        hadn@worker-1:~/multi-container$ kubectl get po
        NAME    READY   STATUS    RESTARTS   AGE
        nginx   4/4     Running   0          2m8s
        ```
