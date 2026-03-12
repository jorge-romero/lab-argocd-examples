# Sync Waves & Hooks Example for ArgoCD

This example demonstrates how to control **deployment order** using Sync Waves and execute **lifecycle hooks** during the sync process.

## Structure

```
sync-waves-hooks/
├── application.yaml                    # ArgoCD Application
└── manifests/
    ├── 00-presync-migration.yaml       # PreSync Hook: DB migration
    ├── 01-wave0-config.yaml            # Wave 0: ConfigMaps & Secrets
    ├── 02-wave1-database.yaml          # Wave 1: Database
    ├── 03-wave2-backend.yaml           # Wave 2: Backend API
    ├── 04-wave3-frontend.yaml          # Wave 3: Frontend + Ingress
    ├── 05-postsync-test.yaml           # PostSync Hook: Smoke tests
    └── 06-syncfail-alert.yaml          # SyncFail Hook: Alert
```

## Deployment Order

```
┌─────────────────────────────────────────────────────────────────┐
│                        SYNC PROCESS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐                                               │
│  │   PreSync    │  → Database migration job                     │
│  │    Hook      │    (runs before anything else)                │
│  └──────┬───────┘                                               │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │   Wave 0     │  → ConfigMaps, Secrets                        │
│  └──────┬───────┘                                               │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │   Wave 1     │  → Database Deployment + Service              │
│  └──────┬───────┘                                               │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │   Wave 2     │  → Backend Deployment + Service               │
│  └──────┬───────┘                                               │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │   Wave 3     │  → Frontend Deployment + Service + Ingress    │
│  └──────┬───────┘                                               │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │  PostSync    │  → Smoke tests, notifications                 │
│  │    Hook      │    (runs after all resources healthy)         │
│  └──────────────┘                                               │
│                                                                 │
│  ┌──────────────┐                                               │
│  │  SyncFail    │  → Alerts (only runs if sync fails)           │
│  │    Hook      │                                               │
│  └──────────────┘                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Deploy

```bash
kubectl apply -f sync-waves-hooks/application.yaml
```

## Sync Waves

Sync waves control the order in which resources are deployed. Lower numbers sync first.

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"   # Deploys first
```

| Wave | Resources | Purpose |
|------|-----------|---------|
| 0 | ConfigMaps, Secrets | Configuration must exist first |
| 1 | Database | Data layer before application |
| 2 | Backend | API before frontend |
| 3 | Frontend, Ingress | User-facing last |

**Important**: Resources in the same wave are synced in parallel. ArgoCD waits for all resources in a wave to be healthy before proceeding to the next wave.

## Hooks

Hooks are Jobs that run at specific points in the sync lifecycle.

### Hook Types

| Hook | When it runs |
|------|--------------|
| `PreSync` | Before any manifests are applied |
| `Sync` | After PreSync, with wave 0 |
| `PostSync` | After all resources are synced and healthy |
| `SyncFail` | When sync operation fails |
| `Skip` | Skips the resource |

### Hook Delete Policies

```yaml
annotations:
  argocd.argoproj.io/hook-delete-policy: HookSucceeded
```

| Policy | Behavior |
|--------|----------|
| `HookSucceeded` | Delete after successful completion |
| `HookFailed` | Delete after failure |
| `BeforeHookCreation` | Delete before new hook is created |

## Common Use Cases

### PreSync Hooks
- Database migrations
- Schema updates
- Backup creation
- Dependency checks

### PostSync Hooks
- Smoke tests / Health checks
- Slack/Teams notifications
- Cache warming
- Documentation updates

### SyncFail Hooks
- PagerDuty/OpsGenie alerts
- Rollback triggers
- Incident creation

## Negative Waves

You can use negative numbers for resources that must be created before wave 0:

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "-1"   # Before wave 0
```

This is useful for CRDs, Namespaces, or RBAC that other resources depend on.
