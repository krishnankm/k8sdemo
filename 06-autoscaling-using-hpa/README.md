# Kubernetes Autoscaling Demo

## Overview
This guide demonstrates a basic Kubernetes Horizontal Pod Autoscaler (HPA) setup using Killerkoda, where the deployment scales automatically based on CPU usage.

## Prerequisites
- A deployed and configured Metrics Server

## Metrics Server
The Kubernetes Metrics Server collects resource metrics from the kubelets and exposes them through the Kubernetes API. These metrics are used by the Horizontal Pod Autoscaler (HPA) to make scaling decisions.

### Install Metrics Server
The Metrics Server is not deployed by default in a Kubernetes cluster. Install it using:
```sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
This deploys the Metrics Server, which will start collecting CPU and memory usage from the nodes.

### Patch Metrics Server (TLS Issue Fix)
Sometimes, the Metrics Server fails due to TLS verification issues because the kubelet uses self-signed certificates. If the Metrics Server cannot verify the kubelet’s certificate, it will not be able to collect metrics. To bypass this, apply the following patch:
```sh
kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
```
Restart the Metrics Server:
```sh
kubectl rollout restart deployment metrics-server -n kube-system
```
This allows the Metrics Server to function properly by ignoring TLS verification errors.

### Verify Metrics Server
To check if the Metrics Server is running:
```sh
kubectl get deployment -n kube-system metrics-server
```
**Note:**
- **Namespace:** A namespace is nothing but a logical separation in Kubernetes where resources are grouped.
- **kube-system:** The kube-system namespace is where the core Kubernetes components, including the Metrics Server, run.

## Horizontal Pod Autoscaler (HPA)
HPA automatically scales the number of pods based on observed CPU or memory usage.
- It ensures that an application has enough replicas to handle the load.
- It also prevents over-provisioning by scaling down when demand decreases.

## Deploy PHP-Apache Server
Here we use the `php-apache` server, a simple web application, to demonstrate autoscaling.
To demonstrate a HorizontalPodAutoscaler, here we will first start a Deployment that runs a container using the hpa-example image, and expose it as a Service using the following manifest:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```
Apply it:
```sh
kubectl apply -f https://k8s.io/examples/application/php-apache.yaml
```

## Create the HorizontalPodAutoscaler 
Now that the server is running, use `kubectl autoscale` to create the autoscaler. This command sets up a HorizontalPodAutoscaler (HPA) for the `php-apache` Deployment, maintaining between 1 and 10 replicas. The HPA controller adjusts the number of replicas to keep average CPU utilization at 50%. This process updates the Deployment, which in turn modifies the ReplicaSet to add or remove Pods accordingly. Since each Pod requests 200 milli-cores, the target average CPU usage is 100 milli-cores. See the algorithm details for more information.

```sh
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```
**Note:**
- `--cpu-percent=50`: The target CPU utilization for scaling.
- `--min=1`: The minimum number of replicas allowed.
- `--max=10`: The maximum number of replicas allowed.

Check HPA status:
```sh
kubectl get hpa
```
Initially, the CPU usage will be 0% since no traffic is being sent to the application.

## Simulate Load
To see autoscaling in action, generate some traffic using a `busybox` pod:
```sh
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
This command continuously sends requests to the `php-apache` service, increasing CPU utilization.

Monitor autoscaling:
```sh
kubectl get hpa php-apache --watch
```
**What happens here?**
- CPU usage will rise, causing the HPA to increase the number of replicas.
- Kubernetes will adjust the number of pods to meet the 50% CPU target.

Check the deployment:
```sh
kubectl get deployment php-apache
```
Watch pod scaling:
```sh
watch -n 1 'kubectl get pods'
```
**Note:** It may take a few minutes for autoscaling to stabilize.

## Stop Load Generation
To stop generating load, terminate the busybox process (`Ctrl + C`).

Verify scaling down:
```sh
kubectl get hpa php-apache --watch
kubectl get deployment php-apache
```
Once CPU utilization drops, HPA automatically reduces the number of replicas back to 1.

## Conclusion
This demo illustrates how Kubernetes HPA dynamically scales applications based on CPU usage. HPA ensures applications can handle varying workloads efficiently by adding or removing pods as needed. This improves resource utilization and optimizes application performance.