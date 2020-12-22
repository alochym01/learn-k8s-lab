# Get all node with 2 columns - NAME and TAINT
- `kubectl get nodes -o=custom-columns='NAME:.metadata.name,TAINT:.spec.taints,NODELABEL:.metadata.labels'`
  ```bash
    NAME       TAINT
    master-1   [map[effect:NoSchedule key:node-role.kubernetes.io/master]]
    worker-1   <none>
  ```
