# Base Nginx Helm Chart

A reusable base Helm chart for deploying nginx-based applications across the infrastructure.

## Overview

This chart provides a production-ready nginx deployment with:
- Security best practices (non-root user, read-only filesystem, dropped capabilities)
- Configurable resource limits and requests
- Health checks (liveness and readiness probes)
- Horizontal Pod Autoscaling support
- Ingress configuration
- Custom configuration via ConfigMap
- Flexible volume mounting

## Installation

### Using as a standalone chart

```bash
helm install my-nginx ./charts/base-nginx
```

### Using with custom values

```bash
helm install my-nginx ./charts/base-nginx -f my-values.yaml
```

### Using as a dependency

Add to your Chart.yaml:

```yaml
dependencies:
  - name: base-nginx
    version: 0.1.0
    repository: "file://../base-nginx"
```

## Configuration

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `2` |
| `image.repository` | Nginx image repository | `nginx` |
| `image.tag` | Image tag (defaults to appVersion) | `""` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container target port | `8080` |
| `resources.limits.cpu` | CPU limit | `200m` |
| `resources.limits.memory` | Memory limit | `256Mi` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `128Mi` |
| `ingress.enabled` | Enable ingress | `false` |
| `autoscaling.enabled` | Enable HPA | `false` |

### Example: Application-specific values

```yaml
# values/my-app.yaml
nameOverride: my-app
replicaCount: 3

image:
  repository: nginx
  tag: "1.27.0-alpine"

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com

configMap:
  enabled: true
  data:
    index.html: |
      <!DOCTYPE html>
      <html>
        <body>
          <h1>My Application</h1>
        </body>
      </html>

volumeMounts:
  - name: config
    mountPath: /usr/share/nginx/html/index.html
    subPath: index.html

volumes:
  - name: config
    configMap:
      name: my-app-base-nginx
```

## Security

The chart implements security best practices:
- Runs as non-root user (UID 101)
- Read-only root filesystem
- Dropped all capabilities
- No privilege escalation

## ArgoCD Integration

Example ArgoCD Application manifest:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-nginx-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/your-repo
    targetRevision: HEAD
    path: charts/base-nginx
    helm:
      valueFiles:
        - values/production.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Maintenance

- Chart version follows SemVer
- appVersion tracks nginx stable releases
- Test deployments before updating chart version
