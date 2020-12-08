# Lab RBAC
## Lab requirements
1.  Create user alochym
    1.  Create public and private key using openSSL
        ```bash
        openssl genrsa -out alochym.key 2048
        ```
    2.  Create Certitcate Request Signing(csr) file
        ```bash
        openssl req -new -key alochym.key -out alochym.csr -subj "/CN=alochym/O=alochym"
        ```
    3.  Encode csr file
        ```bash
        cat alochym.csr | base64 | tr -d '\n'
        ```
    4.  Issue csr file to kubernetes cluster for approving - `can be use kubernetes the hard way`
        1.  `kubectl apply -f alochym-csr.yaml`
        ```yaml
        # alochym-csr.yaml
        apiVersion: certificates.k8s.io/v1beta1
        kind: CertificateSigningRequest
        metadata:
            name: alochym-csr
        spec:
            groups:
            -   system:authenticated
            # request value should be in step 3
            request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1pUQ0NBVTBDQVFBd0lERVFNQTRHQTFVRUF3d0hZV3h2WTJoNWJURU1NQW9HQTFVRUNnd0RiM0puTUlJQgpJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBdFR5YWVpeVVVbEw1REdzaUlNRHlSak9iCkdWRmFsb01IRkZkRGZRMERUVk9zUzlYcmE0U0xhbTV1OFYwQ3IrNW1zRFNvYW1UdEEzbFdFTjE4cjR4aUhDd0cKKzcrTmRVdzg0eFk1QnVxMXI3ZnRXd1gyenRBczk1UVlYTEszNzgzNEgrWXhyTU1WUWJ1VllvMjdRRkJINmVwZAp3c2lIelJzTkk2TGJTKzRGcFF6QUdKVUcwTDZyaXJGS29LSEJGWld1S2dxMXhyUG1EWFNUb0dWZmUyUlpyalhvCnNFaDNVR0lNQTBON3dtd0l6eW1nK2tMaUQrOFJEaG5FdkJvYkZsamFYaFArMVkxTXZNbklxWm9BSHFNdlZMT1EKUUhXTGZMdFFHd3VFMzhnMWwzWVlqRmpjUndyL05aaG9zWVp4K2J2RzEvTlBPcUw5R1FlM2dhL2FZcTl3NXdJRApBUUFCb0FBd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFFUFk5Rm5vQmFkU0p6NTQ1TVB3VGFPYUFwM25KNFdkCjk0RTEvOHBRR3E2eXc3T1FaRXVNdFRGcjh1NzFReUVpTnBuT0FzSXVxVlZ5YVhJV1J4N2taV0J6RXV4Y2hvb2EKc1lkTEVXcDdyenZhOFNWb08zUlM2eHBlQmxLV0JJeDlmY3FoZWZkcWpxWUxzY3VadVJMVWRERnd3RXRhRWY1TQpQZTlKSG5hTmxqRlFsVThNdmFoMlQvQStjZXE5RjNFUjNEUENoL3IwLzk5SUNKUDZhNVlQZFZiS3lyMkZtMzBDClhmUloraExBRG5IdWU0dlN5ZnA3eVpkY1ZTandBb1d5RDBRNnQ2eVFnWnRhbHIxUlZXUjhpSFRyQVpldGJ5K0UKeVVpTEdVZWtxT0ZCUFZBTWVqd0tDcnhPT3JMYU9QbmNEQnpmZG9hYjJ1eXd1RWx0ZW1zbURTZz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
            usages:
            -   digital signature
            -   key encipherment
            -   client auth
        ```
    5.  check issued csr request on kubernetes cluster for approving - `can be use kubernetes the hard way`
        ```bash
        kubectl get csr
        ```
    6.  Approve the csr request - `can be use kubernetes the hard way`
        ```bash
        kubectl certificate approve alochym-csr
        ```
    7.  Download the csr request - `can be use kubernetes the hard way`
        ```bash
        kubectl get csr alochym-csr -o jsonpath='{.status.certificate}' | base64 -d > alochym.crt
        ```
    8.  Create kubeconfig file
        1.  set user
            ```bash
            kubectl config set-credentials alochym --client-certificate=alochym.crt --client-key=alochym.key --embed-certs=true
            ```
        2.  check current context
            ```bash
            kubectl config get-contexts
            ```
        3.  set new context
            ```bash
            kubectl config set-context alochym --user=alochym --namespace=alochym --cluster $cluster_name
            ```
        4.  Switch current context to new context
            ```bash
            kubectl config use-context alochym
            ```
        5.  set cluster server
            ```bash
            kubectl config set-cluster kubernetes --server=https://master-1:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true
            ```
2.  Allow user alochym with get pods permission on alochym namespace
    ```bash
    kubectl get pods
    ```
3.  Create Role object with get pods permission in step 2
    1.  `kubectl apply -f role.yaml`
    ```yaml
    # role.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
        name: alochym
        namespace: alochym
    rules:
    -   apiGroups: ["", "apps"]
        resources: ["*"]
        verbs: ["get", "list", "watch"]
    ```
    2.  check role - `kubectl describe role alochym -n alochym`
4.  Using RoleBinding to set role in step 4 to user alochym
    1.  `kubectl apply -f rolebinding.yaml`
    ```yaml
    # rolebinding.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
        name: alochym-rolebinding
        namespace: alochym
    subjects:
    -   kind: User
        name: alochym
    roleRef:
        kind: Role
        name: alochym
        apiGroup: rbac.authorization.k8s.io
    ```
6.  Check result
    ```bash
    hadn@master-1:~/lab/rbac$ kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    gin-deployment-7b67858ddd-6xzg5   1/1     Running   0          3d7h
    gin-deployment-7b67858ddd-d4jpq   1/1     Running   0          3d7h
    gin-deployment-7b67858ddd-rg4tj   1/1     Running   0          3d7h
    gin-deployment-7b67858ddd-w282t   1/1     Running   0          3d7h
    ```
7.  Reference link
    1.  https://www.howtoforge.com/role-based-access-control-rbac-in-kubernetes/
    2.  https://thenewstack.io/three-realistic-approaches-to-kubernetes-rbac/
    3.  https://www.howtoforge.com/role-based-access-control-rbac-in-kubernetes/
