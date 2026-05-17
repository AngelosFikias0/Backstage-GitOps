# Next Steps — Backstage Platform Roadmap

## Current State
- Backstage running on KinD (local)
- PostgreSQL with persistent storage
- Kubernetes plugin configured
- Guest auth enabled
- Manifests versioned in GitOps repo

---

## Phase 1 — Stabilize Local Platform

### Fix auth 401 (priority)
- Resolve guest auth 403 on `/api/auth/guest/refresh`
- Verify ConfigMap auth block is correctly mounted

### Add NGINX Ingress
- Replace `kubectl port-forward` with proper Ingress
- Access via `http://backstage.local` instead of port-forward
- Required for any realistic traffic routing

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

### Register real catalog entities
- Create `catalog-info.yaml` for your own services
- Push to GitHub
- Register via Backstage UI or add to `catalog.locations` in ConfigMap

---

## Phase 2 — GitOps

### Install ArgoCD
- Deploy ArgoCD into the cluster
- Point it at `Backstage-GitOps` repo
- Any `kubectl apply` becomes a `git push`

```
Git push → ArgoCD detects diff → syncs cluster
```

### Wire CI to GitOps
- GitHub Actions in `Backstage-App` builds image on push
- Pushes to GHCR
- Updates image tag in `Backstage-GitOps`
- ArgoCD auto-syncs

```
Code push → CI builds image → updates GitOps repo → ArgoCD deploys
```

---

## Phase 3 — Platform Hardening

### Secrets management
- Replace plain `postgres-secret.yaml` with Sealed Secrets or External Secrets Operator
- Never commit raw secrets

### Helm chart
- Replace raw manifests with Backstage official Helm chart
- Enables environment overlays (dev/staging/prod values)

```bash
helm repo add backstage https://backstage.github.io/charts
```

### Real auth provider
- Replace guest auth with GitHub OAuth or OIDC
- Required for any multi-user setup

---

## Phase 4 — Extend the Platform

### Software Templates
- Build Scaffolder templates for your org
- Example: "Create a new microservice" template that scaffolds repo + CI + catalog entry

### TechDocs
- Add `mkdocs.yml` to your services
- Backstage renders docs automatically from source

### More Plugins
- GitHub Actions plugin — CI visibility per entity
- Cost Insights — resource cost per service
- Lighthouse — frontend quality audits

---

## Target Architecture

```
Developer → Backstage UI
                ↓
         Software Catalog
         (all services registered)
                ↓
    ┌───────────┼───────────┐
    ↓           ↓           ↓
Kubernetes   TechDocs   Scaffolder
(live state) (docs)    (templates)
    ↓
ArgoCD → KinD Cluster
    ↑
GitHub Actions (CI)
    ↑
Backstage-App repo
```

This is a functional Internal Developer Platform.
