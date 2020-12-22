# Using RBAC for normal user - k8s version 1.19
1. Create normal user "alochym" with permission [create,list,delete,watch] on resource [pods,deployments] of "default" namespace
2. Using "can-i" to check
## Step to work
1. Create role "user-permission"
   ```bash
   kubectl create role user-permission --verb=create,list,delete,watch --resource=pods,deployment --dry-run=client -oyaml |tee role-user-permission.yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     creationTimestamp: null
     name: user-permission
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
2. Create rolebinding "user-binding" to binding role to user "alochym" with "user-permission" role
   ```bash
   kubectl create rolebinding user-binding --role=user-permission --user=alochym -n default --dry-run=client -oyaml |tee rolebinding-user-binding.yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     creationTimestamp: null
     name: user-binding
     namespace: default
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: user-permission
   subjects:
   - apiGroup: rbac.authorization.k8s.io
     kind: User
     name: alochym
   ```
3. Check permission with "can-i"
   1. Syntax: `kubectl auth can-i $action $resource --as $subject`
   ```bash
   kubectl auth can-i get pods -n default --as alochym
   no
   kubectl auth can-i list pods -n default --as alochym
   yes
   kubectl auth can-i list daemonset -n default --as alochym
   no
   ```
4. Reference link
   1. https://www.adaltas.com/en/2019/08/07/users-rbac-kubernetes/
   2. https://cdn2.hubspot.net/hubfs/1665891/Assets/Kubernetes%20Security%20-%20Operating%20Kubernetes%20Clusters%20and%20Applications%20Safely.pdf
