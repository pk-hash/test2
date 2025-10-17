# Infrastructure Operations - ArgoCD

GitOps repository for infrastructure and application deployments managed by ArgoCD.

## 📋 Overview

This repository contains reusable Helm charts and ArgoCD configurations for deploying and managing applications across Kubernetes clusters.

## 📁 Repository Structure

```
.
├── argocd/             # ArgoCD configuration
│   ├── apps.yaml      # Apps application (App of Apps)
│   └── applications/  # Application manifests (production + staging)
├── charts/            # Reusable Helm charts
│   └── base-nginx/   # Base nginx chart for applications
├── examples/         # Environment-specific configurations
│   ├── production/   # Production values
│   ├── staging/      # Staging values
│   └── *.yaml        # Generic examples
└── .github/          # GitHub configuration and Copilot instructions
```

## 🚀 Getting Started

### Prerequisites

- Helm 3.x
- kubectl configured with cluster access
- ArgoCD installed in the cluster (for GitOps deployments)

### Bootstrap ArgoCD (App of Apps Pattern)

Apply the apps application to enable GitOps automation:

```bash
kubectl apply -f argocd/apps.yaml
```

This creates the apps application which automatically manages all applications in `argocd/applications/`.

### Using the Base Nginx Chart

#### Quick Start

```bash
# Install with default values
helm install my-app charts/base-nginx

# Install with production values
helm install my-app charts/base-nginx -f examples/production/api-service.yaml -n production

# Install with staging values
helm install my-app charts/base-nginx -f examples/staging/api-service.yaml -n staging

# Upgrade existing deployment
helm upgrade my-app charts/base-nginx -f my-values.yaml
```

#### ArgoCD Deployment (Recommended)

Add an application manifest to `argocd/applications/`:

```bash
# argocd/applications/my-app.yaml
kubectl apply -f argocd/applications/my-app.yaml

# Or let the apps application auto-discover it:
git add argocd/applications/my-app.yaml
git commit -m "feat(argocd): add my-app application"
git push
```

See the [ArgoCD applications documentation](argocd/applications/README.md) for more details.

See the [base-nginx chart documentation](charts/base-nginx/README.md) for detailed configuration options.

## 🌍 Environments

The repository supports multiple environments with different configurations:

### Production (`production` namespace)
- High availability (3 replicas minimum)
- Aggressive autoscaling (up to 15 replicas)
- Higher resource allocations
- Hostnames: `*.example.com`

**Applications:**
- api-service (api.example.com)
- docs-site (docs.example.com)

### Staging (`staging` namespace)
- Reduced resources for cost efficiency
- Conservative autoscaling (up to 5 replicas)
- Lower replica counts (1-2)
- Hostnames: `*-staging.example.com`

**Applications:**
- api-service-staging (api-staging.example.com)
- docs-site-staging (docs-staging.example.com)

See [examples/README.md](examples/README.md) for detailed environment configurations.

## 📚 Available Charts

### base-nginx

A production-ready, reusable Helm chart for nginx-based deployments.

**Features:**
- Security hardened (non-root, read-only filesystem)
- Configurable resource limits
- Built-in health checks
- Horizontal Pod Autoscaling support
- Ingress configuration
- Custom configuration via ConfigMap

**Quick Example:**

```yaml
# my-app-values.yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.27.0-alpine"

ingress:
  enabled: true
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
```

## 🔄 GitOps Workflow

1. **Create Feature Branch**
   ```bash
   git checkout -b feat/add-my-application
   ```

2. **Make Changes**
   - Add/modify Helm values or manifests
   - Follow conventional commit messages (see below)

3. **Commit Changes**
   ```bash
   git add .
   git commit -m "feat(app): add production deployment for my-service"
   ```

4. **Push and Create PR**
   ```bash
   git push origin feat/add-my-application
   ```

5. **Merge and Deploy**
   - ArgoCD automatically syncs changes after merge to main

## 📝 Commit Convention

This repository uses **Conventional Commits**. All commits must follow this format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types

- `feat`: New feature or infrastructure addition
- `fix`: Bug fix or configuration correction
- `docs`: Documentation changes
- `refactor`: Code refactoring without behavior changes
- `chore`: Maintenance tasks, dependency updates
- `ci`: CI/CD pipeline changes

### Scopes

- `app`: Application manifests
- `cluster`: Cluster configurations
- `argocd`: ArgoCD configurations
- `helm`: Helm charts
- `network`: Network policies
- `monitoring`: Monitoring configs

### Examples

```bash
git commit -m "feat(app): add production deployment for api-service"
git commit -m "fix(helm): correct resource limits in base-nginx"
git commit -m "docs(readme): update deployment procedures"
git commit -m "chore(argocd): upgrade argocd version to 2.10.0"
```

## 🔒 Security Best Practices

- Never commit secrets in plain text
- Use sealed-secrets or external-secrets operator
- All charts run with non-root users
- Resource limits are enforced
- Network policies should be applied per namespace

## 🛠️ Development

### Validating Helm Charts

```bash
# Lint chart
cd charts/base-nginx
helm lint .

# Test template rendering
helm template test-release . --debug

# Dry run installation
helm install --dry-run --debug test-release .
```

### Testing Changes

Before committing changes:

1. Validate YAML syntax
2. Lint Helm charts
3. Test in non-production environment
4. Verify ArgoCD sync status

## 📖 Documentation

- [ArgoCD Apps Application & App of Apps](argocd/applications/README.md)
- [Base Nginx Chart Documentation](charts/base-nginx/README.md)
- [Copilot Instructions](.github/copilot-instructions.md)
- [Example Configurations](examples/)

## 🤝 Contributing

1. Follow the GitOps workflow outlined above
2. Use conventional commits
3. Keep changes atomic and focused
4. Update documentation with your changes
5. Test in non-production first

## 📞 Support

For issues or questions:
- Create an issue in this repository
- Contact the Infrastructure Team

## 📄 License

Internal use only - Infrastructure Team
