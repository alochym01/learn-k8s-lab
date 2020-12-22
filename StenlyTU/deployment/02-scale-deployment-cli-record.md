# Scale hr-app deployment to 3 replicas. Check rollout history
- `kubectl scale deployments hr-app --replicas=3 -n practice --record`
- `kubectl -n practice rollout history deploy hr-app`
