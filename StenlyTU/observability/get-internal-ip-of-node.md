# List the InternalIP of all nodes of the cluster.
- using custom-columns - `kubectl get node -o custom-columns="IP:.status.addresses[0].address, TYPE:.status.addresses[0].type"`
- using jsonpath - `kubectl get node master-1 -o jsonpath='{.status.addresses[0].address}'`


