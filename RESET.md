# Reset n8n Installation (Option A)

## Prerequisites
1. Start Docker Desktop
2. Enable Kubernetes (Settings → Kubernetes → Enable Kubernetes)
3. Wait for Kubernetes to be ready

## Reset Steps

### Step 1: Check Current Installation
```bash
# Check if n8n-test is installed
helm list

# Check what pods are running
kubectl get pods
```

### Step 2: Uninstall Current Release
```bash
# Remove the Helm release (keeps PVCs)
helm uninstall n8n-test
```

### Step 3: Check and Delete PVCs
```bash
# List all PVCs to see what needs to be deleted
kubectl get pvc

# Delete PostgreSQL PVC to reset the database
kubectl delete pvc data-n8n-test-postgresql-0

# Optional: Delete other PVCs for complete reset
kubectl delete pvc data-n8n-test-redis-master-0
kubectl delete pvc n8n-test-data

# Or delete all n8n-test related PVCs at once
kubectl delete pvc -l app.kubernetes.io/instance=n8n-test
```

### Step 4: Verify Cleanup
```bash
# Ensure no PVCs remain
kubectl get pvc

# Ensure no pods remain
kubectl get pods
```

### Step 5: Fresh Installation
```bash
# Navigate to your chart directory
cd /Users/zamboni/repos/wearewebera/n8n-helm-chart

# Install fresh
helm install n8n-test .

# Wait for pods to be ready
kubectl get pods -w
```

### Step 6: Access Fresh n8n
```bash
# Port forward once pods are running
kubectl port-forward service/n8n-test 5678:5678
```

Open: http://localhost:5678

You should now see the **setup wizard** to create your admin account!

## Troubleshooting

If you still see login screen instead of setup:
1. Check n8n logs: `kubectl logs deployment/n8n-test`
2. Verify database is empty: `kubectl exec -it statefulset/n8n-test-postgresql -- psql -U n8n -d n8n -c "SELECT * FROM information_schema.tables WHERE table_schema = 'public';"`

## Alternative: Force Fresh Setup

If the above doesn't work, modify `values.yaml` to add:
```yaml
n8n:
  env:
    N8N_SKIP_SETUP: "false"
    N8N_FORCE_SETUP: "true"
```

Then upgrade:
```bash
helm upgrade n8n-test . --reuse-values
```