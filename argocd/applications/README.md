# ArgoCD Applications

This directory contains ArgoCD Application manifests for the App of Apps pattern.

## Structure

- `../apps.yaml` - Apps application that manages all applications in this directory
- Individual application manifests are placed in this directory

## Usage

### Bootstrap ArgoCD with Apps Application

Apply the apps application to your cluster:

```bash
kubectl apply -f argocd/apps.yaml
```

This will create the apps application which automatically manages all applications defined in this directory.

### Adding New Applications

1. Create a new application manifest in this directory:

```yaml
# argocd/applications/my-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/pk-hash/test2.git
    targetRevision: HEAD
    path: charts/project
    helm:
      valueFiles:
        - ../../examples/frontend-app.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

2. Commit and push:

```bash
git add argocd/applications/my-app.yaml
git commit -m "feat(argocd): add my-app application"
git push
```

3. ArgoCD will automatically detect and deploy the new application.

## App of Apps Pattern

The apps application watches this directory and automatically creates/updates/deletes applications based on the manifests present here. This is known as the "App of Apps" pattern and provides:

- Centralized application management
- GitOps automation for all applications
- Easy onboarding of new applications
- Consistent deployment patterns

## Best Practices

- Use meaningful application names
- Set appropriate sync policies per application
- Use different namespaces for different environments
- Reference specific value files for each application
- Include finalizers for proper cleanup
