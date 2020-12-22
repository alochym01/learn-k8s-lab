# Annotate the pod 'nginx2' to be description='my frontend'.

`kubectl -n practice annotate po nginx2 description='my frontend' --overwrite`
`kubectl -n practice annotate -f 02-nginx-2.yaml description='my frontend'`

**Take-away: use --overwrite when changing labels.**
