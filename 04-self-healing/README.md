# Self-Healing in Kubernetes

Delete a running pod:
```sh
kubectl delete pod <pod-name>
```
### Self-healing Mechanism: When a pod goes down, Kubernetes automatically schedules a new one to replace it.

Or

```sh
kubectl exec -it <pod-name> -- bash -c "kill 1"
```
### Here the pod will get restarted. By default, Kubernetes deployments create pods with `restartPolicy: Always`. This means that if a container inside the pod fails, Kubernetes restarts the container instead of replacing the entire pod.

But now check the pods:
```sh
kubectl get pods
```
Or open a new tab and run the below command to watch the pods:
```sh
watch -n 1 `kubectl get pods`
```

### Kubernetes automatically recreates the pod to maintain stability.
