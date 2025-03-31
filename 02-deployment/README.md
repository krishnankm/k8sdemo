# Using Deployments for Reliability

## Step 1: Create a Deployment
Create a `deployment.yaml` file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

Apply the deployment:
```sh
kubectl apply -f deployment.yaml
```

Verify:
```sh
kubectl get deployments
kubectl get pods
```

### Deployments ensure high availability.
