# Get service's ClusterIP, create a temp busybox-1 pod and 'hit' that IP with wget.
- kubectl -n practice get svc nginx-1
- kubectl -n practice run busybox-1 --rm --image=busybox -it -- sh
- / # wget -O- $CLUSTER_IP:80
