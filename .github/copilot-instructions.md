# Copilot Instructions - ArgoCD Infrastructure Operations

## Repository Purpose
This repository manages infrastructure operations and deployments using ArgoCD. All changes should maintain the declarative GitOps workflow and ensure proper synchronization with deployed resources.

## Commit Convention
**REQUIRED**: Use Conventional Commits specification for all commits.

### Commit Format
```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types
- **feat**: New feature or infrastructure addition
- **fix**: Bug fix or configuration correction
- **docs**: Documentation only changes
- **refactor**: Code/configuration refactoring without changing behavior
- **chore**: Maintenance tasks, dependency updates
- **ci**: CI/CD pipeline changes
- **perf**: Performance improvements
- **test**: Adding or updating tests

### Scopes (examples)
- **app**: Application manifests
- **cluster**: Cluster-level configurations
- **argocd**: ArgoCD-specific configurations
- **helm**: Helm charts and values
- **kustomize**: Kustomize overlays
- **network**: Network policies and configurations
- **storage**: Storage classes and PVCs
- **secrets**: Secret management configurations
- **monitoring**: Monitoring and observability configs

### Examples
```
feat(app): add production deployment for api-service
fix(network): correct ingress path for frontend app
docs(readme): update deployment procedures
chore(argocd): upgrade argocd version to 2.9.0
refactor(helm): consolidate common values
```

## Code Style & Best Practices

### Kubernetes Manifests
- Use declarative YAML configurations
- Follow Kubernetes resource naming conventions (lowercase, hyphens)
- Include proper labels and annotations for ArgoCD tracking
- Always specify resource limits and requests
- Use namespaces to organize resources

### ArgoCD Applications
- Ensure proper `sync-policy` configuration
- Use automated sync with `prune: true` cautiously
- Include health checks for critical applications
- Document sync waves when using phased rollouts

### File Organization
- Group resources by application or service
- Use consistent directory structure
- Keep environment-specific configs separate (dev/staging/prod)
- Use kustomize overlays or Helm values for environment differences

### Security
- Never commit secrets in plain text
- Use sealed-secrets, external-secrets, or similar solutions
- Apply RBAC and network policies appropriately
- Scan manifests for security vulnerabilities before commit

### Version Control
- Keep changes atomic and focused
- Reference issue/ticket numbers in commit body when applicable
- Update relevant documentation with infrastructure changes
- Test changes in non-production before promoting

## Workflow Guidelines
1. Create feature branch from `main`
2. Make infrastructure changes following GitOps principles
3. Validate YAML syntax and Kubernetes schemas
4. Commit with conventional commit message
5. Create PR with clear description of infrastructure impact
6. Wait for ArgoCD sync status before merging
7. Monitor deployments post-merge

## Additional Notes
- All changes trigger ArgoCD synchronization
- Breaking changes should be clearly marked in commit body with `BREAKING CHANGE:`
- Infrastructure changes should be backward compatible when possible
- Document migration steps for breaking changes
