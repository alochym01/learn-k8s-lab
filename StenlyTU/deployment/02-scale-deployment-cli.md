# Scale hr-app deployment to 3 replicas.
- `kubectl scale deployments hr-app --replicas=3 -n practice`
# Update the hr-app image to nginx:1.19.
- `kubectl -n practice set image deploy hr-app nginx=nginx:1.19`
- `kubectl -n practice edit deploy/hr-app`
# Check the rollout history of hr-app and confirm that the replicas are OK.
- `kubectl -n practice rollout history deploy hr-app`
- `kubectl -n practice get deploy hr-app`
- `kubectl -n practice get rs # check that a new replica set has been created`
- `kubectl -n practice get po -l app=hr-app`
# Undo the latest rollout and verify that new pods have the old image (nginx:1.18)
- `kubectl -n practice rollout undo deploy hr-app`
- `kubectl -n practice get po # select one of the 'Running' pods`
- `kubectl -n practice describe po hr-app-695f79495-6gfsw | grep -i Image: # should be nginx:1.18`
# Do an update of the deployment with a wrong image nginx:1.91 and check the status.
- `kubectl -n practice set image deploy/hr-app nginx=nginx:1.91`
- `kubectl -n practice rollout status deploy hr-app`
- `kubectl -n practice get po # you'll see 'ErrImagePull'`
