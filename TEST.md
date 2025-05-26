# Testing n8n Helm Chart on Docker Desktop Kubernetes

## Prerequisites

1. **Enable Kubernetes in Docker Desktop**:
   - Open Docker Desktop
   - Go to Settings → Kubernetes
   - Check "Enable Kubernetes"
   - Click "Apply & Restart"
   - Wait for Kubernetes to start (green status indicator)

2. **Verify Kubernetes is running**:
   ```bash
   kubectl cluster-info
   kubectl get nodes
   ```

## Testing Steps

### Step 1: Validate Chart Structure
```bash
# Lint the chart
helm lint .

# Template generation (without cluster)
helm template n8n-test . --debug
```

### Step 2: Dry-Run Installation
```bash
# Test installation without actually deploying
helm install n8n-test . --dry-run --debug
```

### Step 3: Install for Testing
```bash
# Install the chart
helm install n8n-test .

# Check deployment status
kubectl get pods
kubectl get services
kubectl get pvc
```

### Step 4: Access n8n
```bash
# Port forward to access n8n locally
kubectl port-forward service/n8n-test 5678:5678
```
Then open: http://localhost:5678

**First-Time Access**:
- n8n should show a **setup wizard** to create your admin account
- If you see a login screen instead, the database may have existing data

**If you see login screen instead of setup wizard**:
1. Reset the installation (see "Reset Installation" section below)
2. Check logs: `kubectl logs deployment/n8n-test`

### Step 5: Check Logs
```bash
# Check n8n logs
kubectl logs deployment/n8n-test

# Check PostgreSQL logs
kubectl logs statefulset/n8n-test-postgresql

# Check Redis logs
kubectl logs statefulset/n8n-test-redis-master
```

### Step 6: Test Configuration
```bash
# Check environment variables
kubectl describe pod $(kubectl get pods -l app.kubernetes.io/name=n8n -o jsonpath="{.items[0].metadata.name}")

# Check ConfigMaps and Secrets
kubectl get configmaps
kubectl get secrets
```

### Step 7: Reset Installation (If Needed)

**If you see login screen instead of setup wizard:**

```bash
# 1. Uninstall current release
helm uninstall n8n-test

# 2. Delete PostgreSQL PVC to reset database
kubectl delete pvc data-n8n-test-postgresql-0

# 3. Reinstall fresh
helm install n8n-test .

# 4. Access again
kubectl port-forward service/n8n-test 5678:5678
```

### Step 8: Cleanup
```bash
# Uninstall the release
helm uninstall n8n-test

# Complete cleanup (removes all data)
kubectl delete pvc -l app.kubernetes.io/instance=n8n-test
```

## Troubleshooting

### Common Issues

1. **Pods stuck in Pending**:
   ```bash
   kubectl describe pod <pod-name>
   ```
   - Check if PVCs can be provisioned
   - Check resource limits vs available resources

2. **n8n can't connect to PostgreSQL**:
   ```bash
   kubectl exec -it deployment/n8n-test -- env | grep DB_
   kubectl logs deployment/n8n-test
   ```

3. **Redis connection issues**:
   ```bash
   kubectl exec -it deployment/n8n-test -- env | grep REDIS
   kubectl logs statefulset/n8n-test-redis-master
   ```

4. **Login screen instead of setup wizard**:
   - Database contains existing user data
   - Use reset procedure in Step 7 above
   - Verify fresh database: `kubectl exec -it statefulset/n8n-test-postgresql -- psql -U n8n -d n8n -c "SELECT * FROM public.user;"`

### Resource Requirements
For Docker Desktop, ensure you have allocated sufficient resources:
- **CPU**: 4+ cores recommended
- **Memory**: 8GB+ recommended
- **Storage**: 20GB+ available

### Custom Values for Local Testing
Create `local-values.yaml`:
```yaml
# Smaller resource requirements for local testing
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

# Smaller storage for local testing
n8n:
  persistence:
    size: 2Gi

postgresql:
  primary:
    persistence:
      size: 2Gi

redis:
  master:
    persistence:
      size: 1Gi
```

Install with custom values:
```bash
helm install n8n-test . -f local-values.yaml
```

## Expected Results

After successful installation, you should see:
- 1 n8n pod (Running)
- 1 PostgreSQL pod (Running)
- 1 Redis master pod (Running)
- 3 Redis replica pods (Running)
- All PVCs bound
- Services created and accessible

The n8n interface should be accessible at http://localhost:5678 after port-forwarding.