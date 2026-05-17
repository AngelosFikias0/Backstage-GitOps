# Backstage GitOps

Kubernetes manifests and cluster configuration for the Backstage Internal Developer Portal. Deployment source of truth for the Backstage platform running on KinD.

## Related Repos

| Repo | Purpose |
|---|---|
| [Backstage-App](https://github.com/AngelosFikias0/Backstage-App) | Application source, plugins, templates |
| [Backstage-GitOps](https://github.com/AngelosFikias0/Backstage-GitOps) | Manifests, cluster config, runbook |

## Stack

- **KinD** — local Kubernetes cluster
- **Backstage** — Internal Developer Portal
- **PostgreSQL** — persistent catalog storage
- **NGINX Ingress** — traffic routing (planned)
- **ArgoCD** — GitOps continuous delivery (planned)

## Structure

```
Backstage-GitOps/
├── backstage-cluster-config.yaml   # KinD cluster definition
├── backstage-config.yaml           # Backstage ConfigMap
├── backstage-deployment.yaml       # Backstage Deployment
├── backstage-service.yaml          # Backstage Service
├── backstage-sa.yaml               # ServiceAccount + ClusterRoleBinding
├── postgres-deployment.yaml        # PostgreSQL Deployment
├── postgres-service.yaml           # PostgreSQL Service
├── postgres-pvc.yaml               # PostgreSQL PersistentVolumeClaim
├── catalog-info.yaml               # Backstage catalog entity
├── RUNBOOK.md                      # Operational runbook
└── NEXT_STEPS.md                   # Platform roadmap
```

## Quick Start

See [RUNBOOK.md](./RUNBOOK.md) for full operational procedures.

```bash
# Start existing cluster
docker start backstage-lab-control-plane backstage-lab-worker
kubectl port-forward svc/backstage 7007:7007 -n backstage
```

## Deployment

All resources deploy into the `backstage` namespace.

```bash
kubectl create namespace backstage
kubectl apply -f postgres-secret.yaml      # create manually, never commit
kubectl apply -f postgres-pvc.yaml
kubectl apply -f postgres-deployment.yaml
kubectl apply -f postgres-service.yaml
kubectl apply -f backstage-sa.yaml
kubectl apply -f backstage-config.yaml
kubectl apply -f backstage-deployment.yaml
kubectl apply -f backstage-service.yaml
```

## Secrets

`postgres-secret.yaml` is gitignored. Create it manually:

```bash
kubectl create secret generic postgres-secret \
  --from-literal=POSTGRES_USER=backstage \
  --from-literal=POSTGRES_PASSWORD=backstage \
  --from-literal=POSTGRES_DB=backstage \
  -n backstage
```
