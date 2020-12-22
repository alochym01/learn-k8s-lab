# Use JSON PATH query to retrieve the osImages of all nodes.
- kubectl get nodes master-1 -o jsonpath='{.items[*].status.nodeInfo.osImage}'
# Use JSON PATH query to retrieve the osImages of master-1 node.
- kubectl get nodes master-1 -o jsonpath='{.status.nodeInfo.osImage}'
