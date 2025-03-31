# Deploying a Basic Pod

## Step 1: Create a Pod
Run the following command to deploy an Nginx pod:

```sh
kubectl run nginx --image=nginx
```

Verify the pod is running:

```sh
kubectl get pods
```

### A Pod is the smallest unit in Kubernetes, running a single container.
