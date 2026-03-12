# ArgoCD Examples

A comprehensive collection of ArgoCD examples demonstrating GitOps patterns and best practices for Kubernetes deployments.

## Table of Contents

- [What is ArgoCD?](#what-is-argocd)
- [Core Concepts](#core-concepts)
- [ArgoCD Resources](#argocd-resources)
- [GitOps Workflow](#gitops-workflow)
- [Getting Started](#getting-started)
- [Examples](#examples)

---

## What is ArgoCD?

**ArgoCD** is a declarative, GitOps continuous delivery tool for Kubernetes. It automates the deployment of applications by using Git repositories as the source of truth for defining the desired application state.

### Key Features

| Feature | Description |
|---------|-------------|
| **Declarative** | Application definitions, configurations, and environments are declarative and version controlled |
| **Automated Sync** | Automatically syncs applications to the desired state defined in Git |
| **Self-Healing** | Detects and corrects drift between the desired and live state |
| **Multi-Cluster** | Deploy applications across multiple Kubernetes clusters |
| **SSO Integration** | Supports OIDC, OAuth2, LDAP, SAML 2.0, GitHub, GitLab, and more |
| **Rollback** | Easy rollback to any application state committed in Git |
| **Health Status** | Built-in health assessment for Kubernetes resources |
| **Web UI & CLI** | Rich UI and CLI for managing applications |

---

## Core Concepts

### GitOps Principles

1. **Declarative**: The entire system is described declaratively
2. **Versioned and Immutable**: The canonical desired state is versioned in Git
3. **Pulled Automatically**: Approved changes are automatically applied
4. **Continuously Reconciled**: Software agents ensure correctness and alert on divergence

### ArgoCD Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           ArgoCD Server                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐  │
│  │   API       │    │ Repository  │    │    Application          │  │
│  │   Server    │    │   Server    │    │    Controller           │  │
│  │             │    │             │    │                         │  │
│  │  - Web UI   │    │  - Git ops  │    │  - Monitors apps        │  │
│  │  - CLI      │    │  - Helm     │    │  - Compares state       │  │
│  │  - gRPC/REST│    │  - Kustomize│    │  - Syncs to desired     │  │
│  └─────────────┘    └─────────────┘    └─────────────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
         │                    │                      │
         ▼                    ▼                      ▼
    ┌─────────┐         ┌─────────┐          ┌─────────────┐
    │  Users  │         │   Git   │          │  Kubernetes │
    │         │         │  Repos  │          │   Clusters  │
    └─────────┘         └─────────┘          └─────────────┘
```

---

## ArgoCD Resources

### Application

The **Application** CRD is the core resource that defines:
- Source repository (Git URL, branch, path)
- Destination cluster and namespace
- Sync policy (manual/automated)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/repo.git
    targetRevision: HEAD
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: my-namespace
  syncPolicy:
    automated:
      prune: true      # Delete resources not in Git
      selfHeal: true   # Revert manual changes
```

### ApplicationSet

**ApplicationSet** generates multiple Applications from a single template using generators:

| Generator | Use Case |
|-----------|----------|
| List | Define explicit list of applications |
| Git Directory | Auto-discover from Git directory structure |
| Git File | Generate from JSON/YAML files in Git |
| Cluster | Deploy to multiple clusters |
| Matrix | Combine multiple generators |
| Pull Request | Deploy per pull request |

### AppProject

**AppProject** provides logical grouping and access control:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
  namespace: argocd
spec:
  description: My team's project
  sourceRepos:
    - 'https://github.com/my-org/*'
  destinations:
    - namespace: 'my-namespace'
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
```

---

## GitOps Workflow

### Recommended Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                         GitOps Workflow                              │
└──────────────────────────────────────────────────────────────────────┘

  Developer                Git Repository              ArgoCD & Kubernetes
     │                           │                            │
     │  1. Push changes          │                            │
     │ ─────────────────────────>│                            │
     │                           │                            │
     │                           │  2. Detect changes         │
     │                           │ <─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ │
     │                           │                            │
     │                           │  3. Pull manifests         │
     │                           │ ─────────────────────────> │
     │                           │                            │
     │                           │  4. Compare desired vs live│
     │                           │                       ┌────┴────┐
     │                           │                       │ Compare │
     │                           │                       └────┬────┘
     │                           │                            │
     │                           │  5. Apply changes          │
     │                           │                       ┌────┴────┐
     │                           │                       │  Sync   │
     │                           │                       └────┬────┘
     │                           │                            │
     │  6. View status           │                            │
     │ <─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
     │                           │                            │
```

### Sync Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Manual** | Requires explicit sync trigger | Production, critical environments |
| **Automated** | Syncs automatically on Git changes | Development, staging |
| **Automated + Prune** | Also deletes resources removed from Git | Full GitOps automation |
| **Automated + Self-Heal** | Reverts manual cluster changes | Enforce Git as source of truth |

### Application States

| State | Icon | Description |
|-------|------|-------------|
| **Healthy** | Green | All resources are healthy |
| **Progressing** | Yellow | Resources are being updated |
| **Degraded** | Orange | Some resources are unhealthy |
| **Suspended** | Blue | Application is suspended |
| **Missing** | Red | Resources are missing |
| **Unknown** | Grey | Health status cannot be determined |

---

## Getting Started

### Prerequisites

1. Kubernetes cluster
2. ArgoCD installed ([Installation Guide](https://argo-cd.readthedocs.io/en/stable/getting_started/))
3. `kubectl` and `argocd` CLI tools

### Install ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Deploy Your First Application

```bash
# Using kubectl
kubectl apply -f application.yaml

# Using ArgoCD CLI
argocd app create my-app \
  --repo https://github.com/jorge-romero/lab-argocd-examples.git \
  --path app \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

---

## Examples

This repository contains practical examples for different ArgoCD deployment patterns:

### Basic Deployment Methods

| Example | Description | Link |
|---------|-------------|------|
| **Plain Manifests** | Simple Kubernetes YAML files | [app/](./app/) |
| **Helm Chart** | Templated deployment with Helm | [app-helm/](./app-helm/) |
| **Kustomize** | Overlay-based customization | [app-kustomize/](./app-kustomize/) |

### Advanced Patterns

| Example | Description | Link |
|---------|-------------|------|
| **App of Apps** | Parent application managing child applications | [app-of-apps/](./app-of-apps/) |
| **ApplicationSet** | Generate multiple apps from templates | [app-of-apps-applicationset/](./app-of-apps-applicationset/) |
| **Sync Waves & Hooks** | Control deployment order and lifecycle hooks | [sync-waves-hooks/](./sync-waves-hooks/) |

### Multi-Source

| Example | Description | Link |
|---------|-------------|------|
| **Multi-Repo** | Application with multiple source repositories | [application-multirepo.yaml](./application-multirepo.yaml) |

---

## Quick Reference

### Common ArgoCD CLI Commands

```bash
# Login
argocd login <ARGOCD_SERVER>

# List applications
argocd app list

# Get application details
argocd app get <APP_NAME>

# Sync application
argocd app sync <APP_NAME>

# View application diff
argocd app diff <APP_NAME>

# Rollback to previous version
argocd app rollback <APP_NAME> <HISTORY_ID>

# Delete application
argocd app delete <APP_NAME>
```

### Useful Annotations

```yaml
# Sync wave (deployment order)
argocd.argoproj.io/sync-wave: "1"

# Hook type
argocd.argoproj.io/hook: PreSync|Sync|PostSync|SyncFail|Skip

# Hook delete policy
argocd.argoproj.io/hook-delete-policy: HookSucceeded|HookFailed|BeforeHookCreation

# Compare options
argocd.argoproj.io/compare-options: IgnoreExtraneous

# Sync options
argocd.argoproj.io/sync-options: Prune=false
```

---

## Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [ArgoCD GitHub Repository](https://github.com/argoproj/argo-cd)
- [GitOps Principles](https://opengitops.dev/)
- [CNCF GitOps Working Group](https://github.com/cncf/tag-app-delivery/tree/main/gitops-wg)

---

## License

This project is licensed under the MIT License.
