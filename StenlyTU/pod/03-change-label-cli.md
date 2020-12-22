# Change the labels of pod 'nginx2' to be app=v2.

`kubectl -n practice label po nginx2 app=v2 --overwrite`

**Take-away: use --overwrite when changing labels.**
