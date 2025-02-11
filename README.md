# SysAid GitOps Repository

## Overview
This repository manages SysAid's **multi-tenant, multi-region** GitOps setup using **ArgoCD, Helm, and External Secrets Operator**.

## Repository Structure
```
.gitops-repo/
├── applicationsets/  # ArgoCD ApplicationSets
│   ├── core-tenants.yaml  # Deploys Core instances
│   ├── spaces-regional.yaml  # Deploys Spaces per region
│   ├── cluster-addons.yaml  # Cluster-wide tools
├── argocd-apps.yaml  # Apps of Apps
├── charts/  # Helm charts
├── environments/  # Helm values per environment
├── secrets/  # Encrypted secrets
└── cluster-addons/  # Ingress, Monitoring, etc.
```

## ArgoCD Setup
### Apps of Apps
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: apps-of-apps
  namespace: argocd
spec:
  source:
    repoURL: 'https://github.com/sysaid/gitops-repo'
    targetRevision: main
    path: applicationsets/
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Secrets Management
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: inst04us-secrets
  namespace: inst04us
spec:
  secretStoreRef:
    name: aws-secrets-store
    kind: ClusterSecretStore
  target:
    name: inst04us-secrets
  data:
    - secretKey: db_user
      remoteRef:
        key: global-secrets
        property: db_user
```

## Deployment Workflow
1. **Install ArgoCD:**
   ```sh
   kubectl create namespace argocd
   helm install argocd argo/argo-cd --namespace argocd
   ```
2. **Push configurations to Git:**
   ```sh
   git add . && git commit -m "Initial setup" && git push origin main
   ```
3. **ArgoCD auto-syncs applications.**

## Adding a New Tenant
1. Add a new file: `environments/tenants/instXX-values.yaml`
2. Encrypt secrets: `secrets/tenants/instXX-secrets.yaml.enc`
3. Push changes, and ArgoCD will deploy automatically.

---
🚀 **Fully automated, scalable, and secure!**