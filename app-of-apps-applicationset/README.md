# ApplicationSet Example for ArgoCD

This folder demonstrates using **ApplicationSet** to deploy multiple applications. ApplicationSet is a more powerful and declarative alternative to the App of Apps pattern.

## Structure

```
app-of-apps-applicationset/
├── applicationset.yaml              # Single ApplicationSet resource
└── workloads/                       # Kubernetes manifests
    ├── frontend/
    │   └── deployment.yaml
    ├── backend/
    │   └── deployment.yaml
    └── database/
        └── deployment.yaml
```

## How It Works

The ApplicationSet uses a **List Generator** to define applications:

```yaml
generators:
  - list:
      elements:
        - name: frontend
          namespace: frontend
          path: app-of-apps-applicationset/workloads/frontend
        - name: backend
          namespace: backend
          path: app-of-apps-applicationset/workloads/backend
```

Each element generates an ArgoCD Application using the template.

## Deployment

```bash
kubectl apply -f app-of-apps-applicationset/applicationset.yaml
```

This single command creates:
- `frontend-app` - Deployed to `frontend` namespace
- `backend-app` - Deployed to `backend` namespace
- `database-app` - Deployed to `database` namespace

## Adding a New Application

1. Create manifests in `workloads/<app-name>/`
2. Add an entry to the `generators.list.elements`:
   ```yaml
   - name: newapp
     namespace: newapp
     path: app-of-apps-applicationset/workloads/newapp
   ```
3. Commit and push

## ApplicationSet vs App of Apps

| Feature | ApplicationSet | App of Apps |
|---------|---------------|-------------|
| Single resource | Yes | No (parent + children) |
| Template-based | Yes | No |
| Generator types | List, Git, Cluster, Matrix, etc. | N/A |
| Deletion handling | Automatic | Requires finalizers |
| Complexity | Lower | Higher |

## Other Generator Types

### Git Directory Generator
Automatically discovers directories:
```yaml
generators:
  - git:
      repoURL: https://github.com/jorge-romero/lab-argocd-examples.git
      revision: HEAD
      directories:
        - path: app-of-apps-applicationset/workloads/*
```

### Cluster Generator
Deploy to multiple clusters:
```yaml
generators:
  - clusters: {}
```

### Matrix Generator
Combine multiple generators:
```yaml
generators:
  - matrix:
      generators:
        - list: ...
        - clusters: ...
```
