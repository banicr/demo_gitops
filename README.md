# Demo GitOps Repository

Kubernetes deployment manifests for the Demo Flask App. ArgoCD monitors this repo and automatically deploys changes to the cluster.

## What's Inside

- **Helm Chart** (`helm/demo-flask-app/`) - Kubernetes deployment configuration
- **ArgoCD Application** (`argocd-application.yaml`) - Tells ArgoCD what to deploy

## Quick Start

```bash
# Setup is handled by demo_app setup script
cd ../demo_app/scripts
./setup-local-cluster.sh
```

This creates the cluster, installs ArgoCD, and deploys the app automatically.

## How GitOps Works

```
Code Push → CI/CD Pipeline → Updates this repo → ArgoCD deploys
```

1. You push code to [demo_app](https://github.com/banicr/demo_app)
2. GitHub Actions builds and tests the image
3. Pipeline updates `helm/demo-flask-app/values.yaml` with new image tag
4. ArgoCD detects the change (polls every 3 minutes)
5. ArgoCD deploys to Kubernetes

## Repository Structure

```
demo_gitops/
├── helm/
│   └── demo-flask-app/         # Helm chart
│       ├── Chart.yaml          # Chart metadata
│       ├── values.yaml         # Configuration (updated by CI)
│       └── templates/          # Kubernetes templates
│           ├── deployment.yaml
│           ├── service.yaml
│           └── serviceaccount.yaml
├── argocd-application.yaml     # ArgoCD config
└── README.md
```

## What Gets Updated by CI

When you push code to demo_app, the CI pipeline updates:

**File: `helm/demo-flask-app/values.yaml`**
```yaml
image:
  tag: "c04a0e0-33"         # ← Updated to new version

env:
  appVersion: "c04a0e0-33"  # ← Updated to match
```

## Configuration Files

### Helm Chart

- **Chart.yaml** - Chart name and version
- **values.yaml** - Default configuration (image, replicas, resources)
- **templates/** - Kubernetes manifest templates

### ArgoCD Application

- Points to this GitHub repo
- Monitors `main` branch
- Auto-syncs every 3 minutes
- Deploys to `demo-app` namespace

## Access the Application

```bash
# Port forward the service
kubectl port-forward -n demo-app svc/demo-flask-app 9090:80

# Visit http://localhost:9090
```

## Related Repos

- [demo_app](https://github.com/banicr/demo_app) - Application source code and CI/CD

## ArgoCD

Access ArgoCD UI:
```bash
# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo

# Visit https://localhost:8080
# Username: admin
```
