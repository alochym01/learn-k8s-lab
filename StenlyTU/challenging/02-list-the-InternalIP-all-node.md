# List the InternalIP of all nodes of the cluster
1. `kubectl get nodes -o custom-columns="NAME:.metadata.name,TYPE:.status.addresses[0].type,IPAddress:.status.addresses[0].address"`
