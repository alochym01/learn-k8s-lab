# Use JSON PATH query to retrieve the Images of all pods.
- kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'
# Use JSON PATH query to retrieve the osImages of a pod.
- kubectl get pods busybox-echo -n practice -o jsonpath='{.spec.containers[0].image}'
