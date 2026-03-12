# App of Apps Pattern for ArgoCD

This folder demonstrates the **App of Apps** pattern in ArgoCD. This pattern allows you to manage multiple ArgoCD Applications using a single parent Application.

## Structure

```
app-of-apps/
├── root-application.yaml    # Parent Application (deploy this first)
├── apps/                    # Child Application definitions
│   ├── frontend-app.yaml    # ArgoCD Application for frontend
│   └── backend-app.yaml     # ArgoCD Application for backend
└── workloads/               # Actual Kubernetes manifests
    ├── frontend/
    │   └── deployment.yaml  # Frontend deployment, service, configmap
    └── backend/
        └── deployment.yaml  # Backend deployment, service, configmap
```

## How It Works

1. **Root Application** (`root-application.yaml`): Points to the `apps/` directory
2. **Child Applications** (`apps/*.yaml`): Define individual ArgoCD Applications
3. **Workloads** (`workloads/*/`): Contain the actual Kubernetes resources

## Deployment

### Step 1: Deploy the Root Application

```bash
kubectl apply -f app-of-apps/root-application.yaml
```

This will create the root application in ArgoCD, which will automatically:
- Discover the child applications in `apps/`
- Create the `frontend-app` and `backend-app` Applications
- Deploy the workloads to their respective namespaces

### Step 2: Verify in ArgoCD UI

Access the ArgoCD UI and you should see:
- `root-app` - The parent application
- `frontend-app` - Deployed to `frontend` namespace
- `backend-app` - Deployed to `backend` namespace

## Benefits of App of Apps Pattern

1. **Single Entry Point**: Deploy one Application to manage many
2. **GitOps**: All application definitions are version controlled
3. **Scalability**: Easy to add new applications by adding files to `apps/`
4. **Consistency**: All child apps inherit sync policies and configurations
5. **Automation**: Self-healing and auto-pruning for all managed apps

## Adding a New Application

1. Create your workload manifests in `workloads/<app-name>/`
2. Create an Application definition in `apps/<app-name>-app.yaml`
3. Commit and push - ArgoCD will automatically discover and deploy it

## Customization

Modify the following in each Application:
- `spec.project`: ArgoCD project name
- `spec.source.repoURL`: Your Git repository URL
- `spec.source.targetRevision`: Branch, tag, or commit
- `spec.destination.namespace`: Target Kubernetes namespace
