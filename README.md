# Demo GitOps Repository

Kubernetes deployment manifests for the Demo Flask App. ArgoCD monitors this repo and automatically deploys changes.

## Structure

```
demo_gitops/
├── helm/demo-flask-app/        # Helm chart
│   ├── Chart.yaml              # Chart metadata
│   ├── values.yaml             # Config (updated by CI)
│   └── templates/              # K8s templates
├── argocd-application.yaml     # ArgoCD app definition
└── README.md
```

## What Gets Updated

CI pipeline automatically updates `helm/demo-flask-app/values.yaml`:
```yaml
image:
  tag: "v2.0.0-abc123-45"  # ← New version

env:
  appVersion: "v2.0.0-abc123-45"  # ← Matches image tag
```

ArgoCD detects changes every 3 minutes and syncs to cluster.

## Related

**[demo_app](https://github.com/banicr/demo_app)** - Application code, CI/CD pipeline, setup guide
