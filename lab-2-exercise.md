# Working with ConfigMaps

## Question
- In the ckad-ns2 NameSpace, run a Pod with name alpine-pod. This pod should run a container base on the alpine image
- In this container, two variables must be set:
  - `localport` is set to `localhost:8082`
  - `external_url` is set to `linux.com`

## Solution
1. Create namespace ckad-ns2
   ```bash
   kubectl create ns ckad-ns2 --dry-run=client -o yaml > ckad-ns2.yaml 

   # ckad-ns2.yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     creationTimestamp: null
       name: ckad-ns2
       spec: {}
       status: {}

   kubectl apply -f ckad-ns2.yaml
   namespace/ckad-ns2 created

   kubectl get  ns

   NAME              STATUS   AGE
   ckad-ns2          Active   98s
   default           Active   127d
   kube-public       Active   127d
   kube-system       Active   127d
   ```
2. Create configmaps(cm-ns2) in ckad-ns2 namespace
   ```bash
   kubectl create configmap cm-ns2 -n ckad-ns2 --from-literal=localport=localhost:8082 --from-literal=external_url=linux.com --dry-run=client -o yaml > cm-ns2.yaml

   # cm-ns2.yaml
   apiVersion: v1
   data:
     external_url: linux.com
     localport: localhost:8082
   kind: ConfigMap
   metadata:
     creationTimestamp: null
     name: cm-ns2
     namespace: ckad-ns2

   kubectl apply -f cm-ns2.yaml

   configmap/cm-ns2 created

   kubectl describe cm cm-ns2 -n ckad-ns

   Name:         cm-ns2
   Namespace:    ckad-ns2
   Labels:       <none>
   Annotations:  
   Data
   ====
   external_url:
   ----
   linux.com
   localport:
   ----
   localhost:8082
   Events:  <none>

   ```
3. Create a alpine-pod. Using variables from configmaps(cm-ns2)
   ```bash
   +--------------------------------+--------------------------------+
   | apiVersion: v1                 | apiVersion: v1                 |
   | kind: Pod                      | kind: Pod                      |
   | metadata:                      | metadata:                      |
   |   name: alpine-pod             |   name: alpine-pod             |
   |   namespace: ckad-ns2          |   namespace: ckad-ns2          |
   | spec:                          | spec:                          |
   |   containers:                  |   containers:                  |
   |   - name: alpine-pod           |   - name: alpine-pod           |
   |     image: alpine              |     image: alpine              |
   |     command: [ "env" ]         |     command: [ "env" ]         |
   |     env:                       |     envForm:                   |
   |     - name: localport          |     - configMapRef             |
   |       valueFrom:               |         name: cm-ns2           |
   |         configMapKeyRef:       |   restartPolicy: Never         |
   |           name: cm-ns2         |                                |
   |           key: localport       |                                |
   |     - name: external_url       |                                |
   |       valueFrom:               |                                |
   |         configMapKeyRef:       |                                |
   |           name: cm-ns2         |                                |
   |           key: external_url    |                                |
   |   restartPolicy: Never         |                                |
   +--------------------------------+--------------------------------+
   ```
4. Check result - kubectl logs alpine-pod -n ckad-ns2
   ```bash
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
   HOSTNAME=alpine-pod
   external_url=linux.com
   localport=localhost:8082
   ```
