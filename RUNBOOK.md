# Backstage Platform — Runbook

## Daily Operations

### Start cluster
```bash
docker start backstage-lab-control-plane backstage-lab-worker
kubectl port-forward svc/backstage 7007:7007 -n backstage
```

### Stop cluster
```bash
docker stop backstage-lab-control-plane backstage-lab-worker
```

### Access
```
http://localhost:7007
```

---

## Full Restore (after cluster deletion)

### 1. Recreate cluster
```bash
kind create cluster --config backstage-cluster-config.yaml
```

### 2. Load image
```bash
kind load docker-image backstage-app:1.0.0 --name backstage-lab
```

> If image is missing, rebuild from [Backstage-App](https://github.com/AngelosFikias0/Backstage-App):
> ```bash
> yarn build:backend
> docker build -t backstage-app:1.0.0 -f packages/backend/Dockerfile .
> ```

### 3. Deploy

```bash
kubectl create namespace backstage

# Postgres
kubectl apply -f postgres-secret.yaml
kubectl apply -f postgres-pvc.yaml
kubectl apply -f postgres-deployment.yaml
kubectl apply -f postgres-service.yaml
kubectl rollout status deployment/postgres -n backstage

# Backstage
kubectl apply -f backstage-sa.yaml
kubectl create secret generic backstage-k8s-token \
  --from-literal=K8S_SA_TOKEN=$(kubectl create token backstage-sa -n backstage --duration=8760h) \
  -n backstage
kubectl apply -f backstage-config.yaml
kubectl apply -f backstage-deployment.yaml
kubectl apply -f backstage-service.yaml
kubectl rollout status deployment/backstage -n backstage
```

### 4. Access
```bash
kubectl port-forward svc/backstage 7007:7007 -n backstage
```

---

## Rebuild After Code Changes

```bash
# In Backstage-App repo
yarn build:backend
docker build -t backstage-app:1.0.0 -f packages/backend/Dockerfile .
kind load docker-image backstage-app:1.0.0 --name backstage-lab
kubectl rollout restart deployment/backstage -n backstage
kubectl rollout status deployment/backstage -n backstage
```

---

## Debugging

```bash
# Pod status
kubectl get pods -n backstage

# Backstage logs
kubectl logs -n backstage deployment/backstage --tail=50

# Postgres logs
kubectl logs -n backstage deployment/postgres --tail=50

# Describe pod
kubectl describe pod -n backstage -l app=backstage

# Verify mounted config
kubectl exec -n backstage deployment/backstage -- cat /app/app-config.production.yaml
```
