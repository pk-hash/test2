# Troubleshooting Guide

Common issues and solutions when deploying applications with ArgoCD.

## Issue: "Resource not found in cluster: networking.k8s.io/v1/Ingress"

**Symptoms:**
- ArgoCD shows "Resource not found in cluster" error for Ingress resources
- Application shows as degraded or progressing

**Causes:**
1. Ingress controller not installed in cluster
2. Ingress resource hasn't been created yet (timing issue)
3. ArgoCD sync in progress
4. Wrong Ingress API version

**Solutions:**

### 1. Verify Ingress Controller is Installed

```bash
kubectl get pods -n ingress-nginx
# or
kubectl get ingressclass
```

If not installed, install nginx ingress controller:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

### 2. Check if Ingress is Actually Missing

```bash
# For staging
kubectl get ingress -n staging

# For production  
kubectl get ingress -n production
```

If ingress exists, this is a transient ArgoCD issue - wait for sync to complete.

### 3. Force ArgoCD Sync

```bash
# Via kubectl
kubectl patch application docs-site-staging -n argocd --type merge -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"HEAD"}}}'

# Or use ArgoCD CLI
argocd app sync docs-site-staging

# Or use ArgoCD UI
# Click "Sync" button on the application
```

### 4. Delete and Recreate Application

If the issue persists:

```bash
# Delete the ArgoCD application (keeps resources)
kubectl delete application docs-site-staging -n argocd

# Reapply
kubectl apply -f argocd/applications/docs-site-staging.yaml
```

## Issue: Admission Webhook Denied - Snippet Directives Disabled

**Symptoms:**
- `admission webhook "validate.nginx.ingress.kubernetes.io" denied the request`
- `nginx.ingress.kubernetes.io/configuration-snippet annotation cannot be used`
- `Snippet directives are disabled by the Ingress administrator`

**Cause:**
The nginx ingress controller has snippet directives disabled for security reasons. Using `configuration-snippet` or `server-snippet` annotations will be rejected.

**Solution:**
Remove snippet annotations from ingress configurations:

```yaml
# WRONG - Will be rejected
ingress:
  annotations:
    nginx.ingress.kubernetes.io/configuration-snippet: |
      add_header Cache-Control "public, max-age=3600";

# CORRECT - Use standard annotations
ingress:
  annotations: {}
  # Or use allowed annotations like:
  # nginx.ingress.kubernetes.io/proxy-body-size: "8m"
  # nginx.ingress.kubernetes.io/rate-limit: "100"
```

**Note:** Cache-Control headers should be configured in the nginx container itself, not via ingress annotations. Consider using a custom nginx.conf in a ConfigMap if needed.

## Issue: "Resource not found in cluster: networking.k8s.io/v1/Ingress"

**Symptoms:**
- `MountVolume.SetUp failed for volume "config": configmap "X" not found`

**Solution:**
Ensure ConfigMap name in values matches the Helm release name:

```yaml
# For production (release name: docs-site)
volumes:
  - name: config
    configMap:
      name: docs-site

# For staging (release name: docs-site-staging)
volumes:
  - name: config
    configMap:
      name: docs-site-staging
```

## Issue: Readiness Probe Failed - Connection Refused

**Symptoms:**
- `Readiness probe failed: Get "http://X:Y/": dial tcp X:Y: connect: connection refused`

**Causes:**
- Wrong container port configuration
- Port mismatch between service and container

**Solution:**
Ensure `service.targetPort` matches nginx listening port (usually 80):

```yaml
service:
  type: ClusterIP
  port: 80
  targetPort: 80  # Must match nginx port
```

## Issue: Nginx Cache Directory Errors

**Symptoms:**
- `mkdir() "/var/cache/nginx/client_temp" failed (30: Read-only file system)`

**Solution:**
This is already fixed in base-nginx chart with `nginxCacheVolumes.enabled: true`. Verify it's enabled:

```yaml
nginxCacheVolumes:
  enabled: true  # Should be true by default
```

## General Debugging Commands

```bash
# Check ArgoCD application status
kubectl get application -n argocd
kubectl describe application <app-name> -n argocd

# Check pod status
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>

# Check all resources in namespace
kubectl get all -n staging
kubectl get all -n production

# Check ArgoCD sync status
argocd app get <app-name>
argocd app sync <app-name>
argocd app history <app-name>

# View manifests that would be applied
helm template <release> charts/base-nginx -f examples/<env>/<app>.yaml

# Force recreate pods
kubectl rollout restart deployment/<deployment-name> -n <namespace>
```

## Prevention

1. **Always validate before committing:**
   ```bash
   helm lint charts/base-nginx
   helm template test charts/base-nginx -f examples/staging/api-service.yaml --dry-run
   ```

2. **Test in staging first** before promoting to production

3. **Use sync waves** for resources with dependencies:
   ```yaml
   metadata:
     annotations:
       argocd.argoproj.io/sync-wave: "1"  # Deploy after wave 0
   ```

4. **Monitor ArgoCD notifications** for sync failures

## Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Helm Documentation](https://helm.sh/docs/)
