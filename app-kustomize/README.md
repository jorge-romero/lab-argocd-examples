# Kustomize Example

This folder contains an example of using Kustomize with ArgoCD. This example references the official Kustomize helloWorld example from the kubernetes-sigs repository.

## Structure

```
app-kustomize/
├── application.yaml     # ArgoCD Application pointing to external Kustomize repo
└── apply.sh             # Helper script
```

## ArgoCD Application

The `application.yaml` deploys the Kustomize helloWorld example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kustomize-example
spec:
  project: default
  source:
    path: examples/helloWorld
    repoURL: "https://github.com/kubernetes-sigs/kustomize"
    targetRevision: HEAD
  destination:
    namespace: dev
    server: "https://kubernetes.default.svc"
```

## Deploy

```bash
kubectl apply -f app-kustomize/application.yaml
```

## Kustomize with ArgoCD

### Basic Kustomize Structure

```
my-app/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

### ArgoCD with Overlays

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/your-repo.git
    targetRevision: HEAD
    path: my-app/overlays/prod    # Point to specific overlay
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

### Kustomize Options in ArgoCD

```yaml
source:
  kustomize:
    # Add name prefix/suffix
    namePrefix: prod-
    nameSuffix: -v1
    
    # Override images
    images:
      - nginx:1.25
      - myapp=myregistry/myapp:v2
    
    # Add common labels
    commonLabels:
      environment: production
    
    # Add common annotations
    commonAnnotations:
      managed-by: argocd
```

## When to Use Kustomize

- Multiple environments (dev/staging/prod) with slight variations
- No need for complex templating logic
- Prefer declarative patches over templates
- Already using Kustomize in your workflow
- Want to avoid Helm complexity

## Kustomize vs Helm

| Feature | Kustomize | Helm |
|---------|-----------|------|
| Templating | Patches/overlays | Go templates |
| Learning curve | Lower | Higher |
| Package management | No | Yes |
| Dependencies | Limited | Full support |
| Built into kubectl | Yes | No |
