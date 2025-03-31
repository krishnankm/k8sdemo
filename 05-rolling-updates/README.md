# Rolling Updates & Rollbacks

## Step 1: Perform a Rolling Update
```sh
kubectl set image deployment/nginx-deployment nginx=nginx:1.19
kubectl rollout status deployment/nginx-deployment
```

## Step 2: Rollback if needed
```sh
kubectl rollout undo deployment/nginx-deployment
```

### Watch the pods by running this command in another tab to see the pods getting recreated with the new version and gets recreated again after rollback.
```sh
watch -n 1 `kubectl get pods`
```

### Kubernetes allows updates with zero downtime and easy rollbacks.
