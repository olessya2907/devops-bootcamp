# Lecture 15 — Kubernetes: Commands Reference

## Setup

```bash
# Start cluster
minikube start --driver=docker

# Verify
kubectl get nodes
# → minikube   Ready   control-plane
```

## Deploy the stack

```bash
# From devops-bootcamp root
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/redis-deployment.yaml
kubectl apply -f k8s/backend-deployment.yaml
```

## Check status

```bash
kubectl get pods -n bootcamp
# → backend-xxxx   1/1   Running   0
# → redis-xxxx     1/1   Running   0

kubectl get all -n bootcamp
```

## Access the app (development only)

```bash
kubectl port-forward deployment/backend 3000:3000 -n bootcamp

# In another terminal:
curl localhost:3000/health    # → {"status":"ok"}
curl localhost:3000/info      # → {"hostname":"backend-xxxx",...}
curl localhost:3000/api/hits  # → {"hits":1}
```

## Debug commands

```bash
# Pod details + events
kubectl describe pod -l app=backend -n bootcamp

# Logs
kubectl logs -l app=backend -n bootcamp
kubectl logs -f <pod-name> -n bootcamp   # follow

# Shell into container
kubectl exec -it deployment/backend -n bootcamp -- sh

# Watch pods (auto-refresh)
kubectl get pods -n bootcamp -w
```

## Scale

```bash
kubectl scale deployment backend --replicas=3 -n bootcamp
kubectl scale deployment backend --replicas=1 -n bootcamp
```

## Self-healing demo

```bash
# Delete pod — Deployment recreates it automatically
kubectl delete pod -l app=backend -n bootcamp
kubectl get pods -n bootcamp   # new pod appears
```

## Cleanup

```bash
kubectl delete -f k8s/backend-deployment.yaml
kubectl delete -f k8s/redis-deployment.yaml
kubectl delete -f k8s/namespace.yaml

# Or stop cluster
minikube stop
