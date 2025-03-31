# Self-Healing in Kubernetes

Delete a running pod:
```sh
kubectl delete pod <pod-name>
```

Check again:
```sh
kubectl get pods
```
Or open a new tab and run the below command to watch the pods:
```sh
watch -n 1 `kubectl get pods`
```

### Kubernetes automatically recreates the pod to maintain stability.
