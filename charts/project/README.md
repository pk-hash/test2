# Project Helm Chart

A reusable base Helm chart for deploying applications with nginx and NATS server.

## Overview

This chart provides a production-ready deployment with:
- **Nginx**: Web server for serving static content and reverse proxy
- **NATS**: Lightweight messaging system for microservices communication
- Security best practices (non-root users, read-only filesystem, dropped capabilities)
- Configurable resource limits and requests
- Health checks (liveness and readiness probes)
- Horizontal Pod Autoscaling support
- Ingress configuration for nginx
- Custom configuration via ConfigMap
- Flexible volume mounting

## Installation

### Using as a standalone chart

```bash
helm install my-project ./charts/project
```

### Using with custom values

```bash
helm install my-project ./charts/project -f my-values.yaml
```

### Using as a dependency

Add to your Chart.yaml:

```yaml
dependencies:
  - name: project
    version: 0.1.0
    repository: "file://../project"
```

## Configuration

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `2` |
| `nginx.enabled` | Enable nginx container | `true` |
| `nginx.image.repository` | Nginx image repository | `nginx` |
| `nginx.image.tag` | Nginx image tag | `1.27.0` |
| `nginx.service.port` | Nginx service port | `80` |
| `nginx.resources.limits.cpu` | Nginx CPU limit | `200m` |
| `nginx.resources.limits.memory` | Nginx memory limit | `256Mi` |
| `nats.enabled` | Enable NATS container | `true` |
| `nats.image.repository` | NATS image repository | `nats` |
| `nats.image.tag` | NATS image tag | `2.10.20-alpine` |
| `nats.service.client.port` | NATS client port | `4222` |
| `nats.service.monitoring.port` | NATS monitoring port | `8222` |
| `nats.resources.limits.cpu` | NATS CPU limit | `200m` |
| `nats.resources.limits.memory` | NATS memory limit | `256Mi` |
| `ingress.enabled` | Enable ingress | `false` |
| `autoscaling.enabled` | Enable HPA | `false` |

### Example: Application-specific values

```yaml
# values/my-app.yaml
nameOverride: my-app
replicaCount: 3

nginx:
  enabled: true
  image:
    tag: "1.27.0-alpine"
  resources:
    limits:
      cpu: 500m
      memory: 512Mi

nats:
  enabled: true
  resources:
    limits:
      cpu: 300m
      memory: 384Mi

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
```

### Example: Disable NATS

If you only need nginx:

```yaml
nginx:
  enabled: true

nats:
  enabled: false
```

### Example: Disable Nginx

If you only need NATS:

```yaml
nginx:
  enabled: false

nats:
  enabled: true
```

## Security

The chart implements security best practices:
- Runs as non-root users (nginx: UID 101, NATS: UID 1000)
- Read-only root filesystem for both containers
- Dropped all capabilities
- No privilege escalation
- Automatic writable volumes for nginx cache directories (/var/cache/nginx, /var/run)

## NATS Configuration

NATS is configured with:
- Client port: 4222 (for client connections)
- Monitoring port: 8222 (for HTTP monitoring and health checks)
- Health checks via monitoring endpoint

To connect to NATS from your application:
```
nats://<service-name>:4222
```

## ArgoCD Integration

Example ArgoCD Application manifest:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-project-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/your-repo
    targetRevision: HEAD
    path: charts/project
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
- Test deployments before updating chart version
- Both nginx and NATS can be independently enabled/disabled
