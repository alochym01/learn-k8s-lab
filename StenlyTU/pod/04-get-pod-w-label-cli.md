# Get only pods with label 'app=v2' from all namespaces.

- `kubectl get pods -A -l app=v2`
- `kubectl get pods --all-namespaces=true -l app=v2`

**Take-away: -l can be used to filter resources by labels**
