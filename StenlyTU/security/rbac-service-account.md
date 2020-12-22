# Using RBAC for service account - k8s version 1.19
1. Create service account "sa-alochym" with permission [create,list,delete,watch] on resource [pods,deployments] of "default" namespace
2. Using "can-i" to check
## Step to work
1. Create service account in default namespace
   ```bash
   kubectl create serviceaccount sa-alochym -n default --dry-run=client -oyaml | tee service-account.yaml


   apiVersion: v1
   kind: ServiceAccount
   metadata:
     creationTimestamp: null
     name: sa-alochym
     namespace: default
   ```
2. Create role "serviceaccount-permission"
   ```bash
   kubectl create role serviceaccount-permission --verb=create,list,delete,watch --resource=pods,deployment --dry-run=client -oyaml |tee role-serviceaccount-permission.yaml


   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     creationTimestamp: null
     name: serviceaccount-permission
   rules:
   - apiGroups:
     - ""
     resources:
     - pods
     verbs:
     - create
     - list
     - delete
     - watch
   - apiGroups:
     - apps
     resources:
     - deployments
     verbs:
     - create
     - list
     - delete
     - watch
   ```
3. Create rolebinding "serviceaccount-binding" to binding role to user "sa-alochym" with "serviceaccount-permission" role
   ```bash
   kubectl create rolebinding serviceaccount-binding --role=serviceaccount-permission --user=default:alochym -n default --dry-run=client -oyaml |tee rolebinding-serviceaccount-binding.yaml


   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     creationTimestamp: null
     name: serviceaccount-binding
     namespace: default
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: serviceaccount-permission
   subjects:
   - apiGroup: rbac.authorization.k8s.io
     kind: ServiceAccount
     name: sa-alochym
   ```
4. Check permission with "can-i"
   1. Syntax: `kubectl auth can-i $action $resource --as namespace:$subject`
   ```bash
   kubectl auth can-i get pods --as default:sa-alochym
   no
   kubectl auth can-i list pods --as default:sa-alochym
   yes
   kubectl auth can-i list daemonset --as default:sa-alochym
   no
   ```
5. Reference link
   1. https://www.adaltas.com/en/2019/08/07/users-rbac-kubernetes/
   2. https://cdn2.hubspot.net/hubfs/1665891/Assets/Kubernetes%20Security%20-%20Operating%20Kubernetes%20Clusters%20and%20Applications%20Safely.pdf
