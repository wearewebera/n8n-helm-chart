# n8n Deployment Fix

## Issues Fixed

1. **N8N_PORT Error**: Added N8N_PORT environment variable to deployment.yaml
2. **Redis Connection**: Ensured proper Redis configuration for queue mode
3. **File Permissions**: Added N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS setting

## Deployment Steps

### Option 1: Deploy from Updated Local Chart

```bash
# 1. Generate secure passwords
export REDIS_PASSWORD=$(openssl rand -base64 32)
export POSTGRES_PASSWORD=$(openssl rand -base64 32)
export N8N_PASSWORD=$(openssl rand -base64 32)
export ENCRYPTION_KEY=$(openssl rand -hex 16)

# 2. Update n8n-values.yaml with your passwords
sed -i "s/your-secure-redis-password/$REDIS_PASSWORD/g" n8n-values.yaml
sed -i "s/your-secure-postgres-password/$POSTGRES_PASSWORD/g" n8n-values.yaml
sed -i "s/your-secure-n8n-password/$N8N_PASSWORD/g" n8n-values.yaml
sed -i "s/your-32-character-encryption-key/$ENCRYPTION_KEY/g" n8n-values.yaml

# 3. Update dependencies
helm dependency update

# 4. Deploy
helm install n8n . -f n8n-values.yaml
```

### Option 2: Deploy with Inline Values (Quick Fix)

```bash
helm install n8n oci://ghcr.io/wearewebera/n8n --version 1.95.3-1 \
  --set n8n.env.N8N_PORT="5678" \
  --set n8n.env.N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS="true" \
  --set n8n.env.QUEUE_BULL_REDIS_PORT="6379" \
  --set n8n.encryption_key="$(openssl rand -hex 16)" \
  --set redis.auth.password="$(openssl rand -base64 32)" \
  --set postgresql.auth.postgresPassword="$(openssl rand -base64 32)" \
  --set postgresql.auth.password="$(openssl rand -base64 32)"
```

## Verify Deployment

```bash
# Check pod status
kubectl get pods -l app.kubernetes.io/name=n8n

# Check logs
kubectl logs -l app.kubernetes.io/name=n8n

# Port forward to access n8n
kubectl port-forward service/n8n 5678:5678
```

Then access n8n at http://localhost:5678

## Troubleshooting

If you still see Redis connection errors:
```bash
# Check Redis pod
kubectl get pods -l app.kubernetes.io/name=redis

# Check Redis service
kubectl get svc | grep redis

# Test Redis connection
kubectl exec -it deployment/n8n -- redis-cli -h n8n-redis-master ping
```