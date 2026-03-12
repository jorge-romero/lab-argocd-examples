# Helm Chart Example

This folder contains a Helm chart for deploying an application with ArgoCD. Helm provides templating and package management for Kubernetes applications.

## Structure

```
app-helm/
├── Chart.yaml           # Chart metadata
├── values.yaml          # Default values
├── .helmignore          # Files to ignore
└── templates/
    ├── app-cm.yaml      # ConfigMap template
    ├── app-deployment.yaml  # Deployment template
    ├── app-ingress.yaml     # Ingress template
    └── app-svc.yaml         # Service template
```

## Chart Info

| Field | Value |
|-------|-------|
| Name | app-helm |
| Version | 0.1.0 |
| App Version | 1.0.0 |
| Type | application |

## Values

```yaml
metadata:
  labels:
    app: argocdexample-helm
  name: argocdexample-helm
url: argocdexample-helm.lab
```

## ArgoCD Application

See `application-helm.yaml` in the root directory, or create one:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: helm-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/jorge-romero/lab-argocd-examples.git
    targetRevision: HEAD
    path: app-helm
    helm:
      releaseName: my-app
      # Override values
      values: |
        metadata:
          name: custom-name
      # Or use valueFiles
      # valueFiles:
      #   - values-prod.yaml
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
# Using ArgoCD CLI
argocd app create helm-app \
  --repo https://github.com/jorge-romero/lab-argocd-examples.git \
  --path app-helm \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --helm-set metadata.name=my-release
```

## Helm with ArgoCD Features

### Override Values

```yaml
source:
  helm:
    values: |
      replicas: 3
      image:
        tag: v2.0.0
```

### Use External Values File

```yaml
source:
  helm:
    valueFiles:
      - values-production.yaml
```

### Set Individual Parameters

```yaml
source:
  helm:
    parameters:
      - name: image.tag
        value: "v1.2.3"
      - name: replicas
        value: "5"
```

## When to Use Helm

- Complex applications with many resources
- Need for templating and conditionals
- Multiple environments with different configurations
- Reusable charts across projects
- Using charts from public Helm repositories
