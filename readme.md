# Vite Project Kubernetes Deployment with Horizontal Pod Autoscaling

This project demonstrates how to deploy a Vite application on Kubernetes with **Horizontal Pod Autoscaling (HPA)** and **NodePort Service** using Minikube. It includes step-by-step instructions to deploy, expose, and test auto-scaling based on CPU usage.

---

## Table of Contents

* [Project Overview](#project-overview)
* [Prerequisites](#prerequisites)
* [Kubernetes Resources](#kubernetes-resources)
* [Deployment Steps](#deployment-steps)
* [Testing Autoscaling](#testing-autoscaling)
* [Observing HPA and Pods](#observing-hpa-and-pods)
* [Cleanup](#cleanup)

---

## Project Overview

This project uses the following Kubernetes resources:

1. **Namespace** – `horizontal-scaling`
2. **Deployment** – `vite-project` (3 replicas, RollingUpdate strategy)
3. **Horizontal Pod Autoscaler (HPA)** – Scales between 1-10 pods based on CPU utilization
4. **Service** – Exposes the deployment as NodePort on port `30021`

Autoscaling is triggered when the CPU utilization exceeds 25%.

---

## Prerequisites

* [Minikube](https://minikube.sigs.k8s.io/docs/start/) installed and running
* [kubectl](https://kubernetes.io/docs/tasks/tools/) installed
* Docker (for building images, if needed)
* Minikube metrics server enabled:

```bash
minikube addons enable metrics-server
```

---

## Kubernetes Resources

### 1. Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: horizontal-scaling
```

### 2. Deployment (`vite-project`)

* 3 replicas
* RollingUpdate strategy (`maxSurge: 0`, `maxUnavailable: 1`)
* CPU/memory requests and limits specified

### 3. Horizontal Pod Autoscaler (`nginx-hpa`)

* Monitors CPU utilization
* Scales between 1 to 10 pods
* Target CPU utilization: 25%

### 4. Service

* Type: NodePort
* Exposes port `80` internally and `30021` externally

---

## Deployment Steps

1. Apply the namespace:

```bash
kubectl apply -f namespace.yaml
```

2. Deploy the Vite app:

```bash
kubectl apply -f nginx-deployment.yaml
```

3. Apply the Horizontal Pod Autoscaler:

```bash
kubectl apply -f nginx-hpa.yaml
```

4. Expose the deployment via NodePort Service:

```bash
kubectl apply -f service.yaml
```

---

## Testing Autoscaling

Generate CPU load to trigger HPA:

```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never \
  -- /bin/sh -c "while true; do wget -q -O- http://vite-project; done" \
  -n horizontal-scaling
```

---

## Observing HPA and Pods

Monitor the HPA scaling behavior:

```bash
kubectl get hpa -w
```

Watch pods scale up or down:

```bash
kubectl get pods -w -n horizontal-scaling
```

Check CPU usage of nodes:

```bash
kubectl top nodes
```

---

## Cleanup

Remove all resources after testing:

```bash
kubectl delete namespace horizontal-scaling
kubectl delete pod load-generator
```

---

## Notes

* Make sure the labels in the deployment and service match exactly for proper service discovery.
* Adjust CPU target utilization in HPA as per your application's load requirements.
* RollingUpdate strategy ensures zero downtime during deployment updates.
