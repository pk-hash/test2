# Example Values Files

This directory contains environment-specific value files for deploying applications using the base-nginx chart.

## Directory Structure

```
examples/
├── production/          # Production environment values
│   ├── api-service.yaml
│   └── docs-site.yaml
├── staging/            # Staging environment values
│   ├── api-service.yaml
│   └── docs-site.yaml
├── frontend-app.yaml   # Generic frontend example
├── static-site.yaml    # Static site example
└── argocd-application.yaml  # ArgoCD application template
```

## Environments

### Production

- Higher resource allocations
- More replicas (HA)
- Aggressive autoscaling
- Namespace: `production`
- Hostnames: `*.example.com`

**Examples:**
- `production/api-service.yaml` - API service with 3 replicas, scales 3-15
- `production/docs-site.yaml` - Documentation site with 2 replicas

### Staging

- Reduced resource allocations
- Fewer replicas
- Conservative autoscaling
- Namespace: `staging`
- Hostnames: `*-staging.example.com`

**Examples:**
- `staging/api-service.yaml` - API service with 2 replicas, scales 2-5
- `staging/docs-site.yaml` - Documentation site with 1 replica

## Usage

### Deploy to Production

```bash
helm install api-service charts/base-nginx -f examples/production/api-service.yaml -n production
```

### Deploy to Staging

```bash
helm install api-service charts/base-nginx -f examples/staging/api-service.yaml -n staging
```

### Via ArgoCD

Production and staging applications are automatically deployed via the apps application (App of Apps pattern):

```bash
kubectl apply -f argocd/apps.yaml
```

## Key Differences Between Environments

| Aspect | Production | Staging |
|--------|-----------|---------|
| Replicas | 2-3 (base) | 1-2 (base) |
| CPU Limits | 200m-500m | 100m-200m |
| Memory Limits | 256Mi-512Mi | 128Mi-256Mi |
| Autoscaling Max | 10-15 | 5 |
| Rate Limits | Higher | Lower |
| Cache TTL | Longer | Shorter |

## Creating New Environments

To create a new environment (e.g., `dev`):

1. Create directory: `examples/dev/`
2. Copy and modify values files
3. Update hostnames and resource limits
4. Create ArgoCD applications in `argocd/applications/`
5. Set appropriate namespace

Example ArgoCD application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: api-service-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/pk-hash/test2.git
    targetRevision: HEAD
    path: charts/base-nginx
    helm:
      valueFiles:
        - ../../examples/dev/api-service.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
