# Plain Kubernetes Manifests Example

This folder contains a simple application deployed using plain Kubernetes YAML manifests. This is the most basic way to deploy applications with ArgoCD.

## Structure

```
app/
├── app-cm.yaml          # ConfigMap with HTML content
├── app-deployment.yaml  # Nginx Deployment
├── app-ingress.yaml     # Ingress resource
└── app-svc.yaml         # Service
```

## Resources

| Resource | Name | Description |
|----------|------|-------------|
| ConfigMap | website-cm | HTML content served by nginx |
| Deployment | website-deployment | Nginx container serving static content |
| Service | website-svc | ClusterIP service |
| Ingress | website-ingress | External access |

## Create ArgoCD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: plain-manifests-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/jorge-romero/lab-argocd-examples.git
    targetRevision: HEAD
    path: app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Deploy

```bash
# Option 1: Apply the Application manifest
kubectl apply -f application.yaml

# Option 2: Using ArgoCD CLI
argocd app create plain-manifests-app \
  --repo https://github.com/jorge-romero/lab-argocd-examples.git \
  --path app \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

## When to Use Plain Manifests

- Simple applications with few resources
- Learning Kubernetes/ArgoCD
- When you don't need templating or environment-specific values
- Quick prototyping
