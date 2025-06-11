# n8n Helm Chart

A Helm chart for deploying [n8n](https://n8n.io/) workflow automation platform with PostgreSQL and Redis on Kubernetes.

## Features

- 🚀 **Complete n8n deployment** with PostgreSQL and Redis
- 📦 **OCI-compatible** Helm chart published to GitHub Container Registry
- 🔧 **Configurable** resources, persistence, and scaling
- 🌐 **Ingress support** for external access
- 📈 **Horizontal Pod Autoscaling** support
- 🔒 **Security** contexts and service accounts
- 💾 **Persistent storage** for n8n data and databases

## Quick Start

### Prerequisites

- Kubernetes 1.19+
- Helm 3.8+

### Installation

#### From GitHub Container Registry (Recommended)

```bash
# Install latest version
helm install n8n oci://ghcr.io/wearewebera/n8n --version 1.95.3-4

# Or with custom values
helm install n8n oci://ghcr.io/wearewebera/n8n --version 1.95.3-4 -f custom-values.yaml
```

#### From Source

```bash
# Clone the repository
git clone https://github.com/wearewebera/n8n-helm-chart.git
cd n8n-helm-chart

# Update dependencies
helm dependency update

# Install the chart
helm install n8n . -f values.yaml
```

### Accessing n8n

By default, n8n is accessible via ClusterIP service on port 5678. To access it:

#### Port Forward (Development)

```bash
kubectl port-forward service/n8n 5678:5678
```

Then open http://localhost:5678

#### Ingress (Production)

Enable ingress in `values.yaml`:

```yaml
ingress:
  enabled: true
  className: 'nginx' # or your ingress class
  hosts:
    - host: n8n.yourdomain.com
      paths:
        - path: /
          pathType: Prefix
```

#### Tailscale Ingress (Secure Internal Access)

For secure access via [Tailscale](https://tailscale.com/kb/1439/kubernetes-operator-cluster-ingress):

```yaml
ingress:
  tailscale:
    enabled: true
    hostname: "n8n"  # Optional: custom hostname (defaults to release-namespace)
    funnel: false    # Optional: expose to public internet via Tailscale Funnel
    proxyGroup: ""   # Optional: HA proxy group (requires Tailscale 1.84+)
```

Access via: `https://n8n.your-tailnet.ts.net` (or your custom hostname)

**Note**: When Tailscale ingress is enabled, it takes precedence over standard ingress configuration.

## Configuration

### Key Configuration Options

| Parameter                 | Description               | Default                        |
| ------------------------- | ------------------------- | ------------------------------ |
| `replicaCount`            | Number of n8n replicas    | `1`                            |
| `image.repository`        | n8n image repository      | `n8nio/n8n`                    |
| `image.tag`               | n8n image tag             | `1.95.3`                       |
| `n8n.encryption_key`      | n8n encryption key        | `change-me-to-a-random-string` |
| `n8n.persistence.enabled` | Enable persistent storage | `true`                         |
| `n8n.persistence.size`    | Storage size              | `8Gi`                          |
| `postgresql.enabled`      | Enable PostgreSQL         | `true`                         |
| `redis.enabled`           | Enable Redis              | `true`                         |

### Example Custom Values

```yaml
# Production configuration example
replicaCount: 2

resources:
  limits:
    cpu: 2000m
    memory: 2Gi
  requests:
    cpu: 1000m
    memory: 1Gi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

ingress:
  enabled: true
  className: 'nginx'
  annotations:
    cert-manager.io/cluster-issuer: 'letsencrypt-prod'
  hosts:
    - host: n8n.yourdomain.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: n8n-tls
      hosts:
        - n8n.yourdomain.com

n8n:
  encryption_key: 'your-super-secret-encryption-key-here'
  env:
    WEBHOOK_URL: 'https://n8n.yourdomain.com'
    GENERIC_TIMEZONE: 'America/New_York'
```

## ArgoCD Deployment

### Important: Password Auto-generation Issue

⚠️ **ArgoCD users must provide explicit passwords** for PostgreSQL, Redis, and n8n encryption key. The auto-generation feature using `randAlphaNum` generates different passwords on each sync, causing authentication failures.

### Step 1: Generate Secure Passwords

```bash
# Generate passwords once and save them securely
export POSTGRES_ADMIN_PASSWORD=$(openssl rand -base64 32)
export POSTGRES_USER_PASSWORD=$(openssl rand -base64 32)
export REDIS_PASSWORD=$(openssl rand -base64 32)
export N8N_ENCRYPTION_KEY=$(openssl rand -hex 16)

echo "PostgreSQL Admin: $POSTGRES_ADMIN_PASSWORD"
echo "PostgreSQL User: $POSTGRES_USER_PASSWORD"
echo "Redis: $REDIS_PASSWORD"
echo "n8n Encryption: $N8N_ENCRYPTION_KEY"
```

### Step 2: Create ArgoCD Values File

```yaml
# argocd-values.yaml
postgresql:
  auth:
    postgresPassword: "your-generated-postgres-admin-password"
    password: "your-generated-postgres-user-password"

redis:
  auth:
    password: "your-generated-redis-password"

n8n:
  encryption_key: "your-generated-n8n-encryption-key"
  
  # Optional: Configure ingress
  # env:
  #   WEBHOOK_URL: "https://n8n.yourdomain.com"
```

### Step 3: ArgoCD Application Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: n8n
  namespace: argocd
spec:
  project: default
  source:
    repoURL: ghcr.io/wearewebera
    targetRevision: 1.95.3-4
    chart: n8n
    helm:
      values: |
        postgresql:
          auth:
            postgresPassword: "your-generated-postgres-admin-password"
            password: "your-generated-postgres-user-password"
        redis:
          auth:
            password: "your-generated-redis-password"
        n8n:
          encryption_key: "your-generated-n8n-encryption-key"
        ingress:
          enabled: true
          className: nginx
          hosts:
            - host: n8n.yourdomain.com
              paths:
                - path: /
                  pathType: Prefix
  destination:
    server: https://kubernetes.default.svc
    namespace: n8n
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### Alternative: Using Existing Secrets

If you prefer to manage secrets separately:

1. Create secrets manually (using the expected names for your release):
```bash
# For release name "n8n", the chart expects these secret names:
kubectl create secret generic n8n-postgresql \
  --from-literal=postgres-password="$POSTGRES_ADMIN_PASSWORD" \
  --from-literal=password="$POSTGRES_USER_PASSWORD" \
  -n n8n

kubectl create secret generic n8n-redis \
  --from-literal=redis-password="$REDIS_PASSWORD" \
  -n n8n

kubectl create secret generic n8n-encryption \
  --from-literal=encryption-key="$N8N_ENCRYPTION_KEY" \
  -n n8n
```

2. Reference the custom secret in ArgoCD values (for n8n encryption only):
```yaml
# The PostgreSQL and Redis subcharts will create their own secrets
# named n8n-postgresql and n8n-redis automatically
postgresql:
  auth:
    postgresPassword: "your-generated-postgres-admin-password"
    password: "your-generated-postgres-user-password"

redis:
  auth:
    password: "your-generated-redis-password"

# Only n8n supports using an existing secret
n8n:
  existingSecret: n8n-encryption
```

**Note**: The Bitnami PostgreSQL and Redis subcharts don't use `existingSecret` at the root level. They create secrets named `{release-name}-postgresql` and `{release-name}-redis` automatically.

### Important Notes for ArgoCD

- **Never rely on auto-generated passwords** with ArgoCD
- **Store passwords securely** in your secret management system
- **Consider using** Sealed Secrets, SOPS, or Vault for production
- **Backup your encryption key** - losing it means losing access to all n8n data

## Testing

This Helm chart includes comprehensive unit tests using [helm-unittest](https://github.com/helm-unittest/helm-unittest).

### Prerequisites

Install the helm-unittest plugin:

```bash
helm plugin install https://github.com/helm-unittest/helm-unittest.git
```

### Running Tests

```bash
# Run all tests
helm unittest .

# Run tests with verbose output
helm unittest --color -v .

# Run specific test file
helm unittest -f 'tests/deployment_test.yaml' .

# Generate JUnit XML output (useful for CI/CD)
helm unittest -t junit -o test-results.xml .
```

### Test Coverage

The test suite covers all major components:

- ✅ **Deployment** - Image, replicas, security context, resources
- ✅ **Service** - Type, ports, labels, selectors
- ✅ **ServiceAccount** - Creation, annotations, automount
- ✅ **Ingress** - Hosts, TLS, annotations, API versions
- ✅ **HPA** - CPU/memory metrics, scaling configuration
- ✅ **PVC** - Storage size, access modes, storage class
- ✅ **Secret** - Encryption key handling

See the [tests README](tests/README.md) for detailed information about the test suite.

### CI/CD Integration

The chart includes GitHub Actions workflow for automated testing:

- Helm chart linting
- Unit tests with helm-unittest
- Integration tests with chart-testing
- Security scanning with Trivy

## Publishing Your Own Version

### Prerequisites for Publishing

1. **GitHub Repository**: Fork or create your own repository
2. **GitHub Token**: Create a Personal Access Token with `write:packages` scope
3. **Repository Settings**: Ensure GitHub Actions are enabled

### Publishing Steps

1. **Update Chart Version**: Modify `version` in `Chart.yaml`
2. **Create GitHub Release**:
   - Go to GitHub → Releases → Create a new release
   - Set tag version (e.g., `v1.95.3-2`)
   - Add release notes describing changes
   - Click "Publish release"
3. **GitHub Action**: The workflow automatically triggers and publishes to GitHub Container Registry

### Versioning Strategy

This chart uses **App Version + Chart Revision** format:

- **Chart Version**: `{n8n-version}-{chart-revision}` (e.g., `1.95.3-1`)
- **App Version**: n8n version (e.g., `1.95.3`)

Examples:
- `1.95.3-1` - First chart release for n8n 1.95.3
- `1.95.3-2` - Chart bug fix for n8n 1.95.3
- `1.96.0-1` - First chart release for n8n 1.96.0

This approach clearly shows both the n8n version and chart iteration.

### Manual Publishing

```bash
# Package the chart
helm dependency update
helm package .

# Login to GitHub Container Registry
helm registry login ghcr.io -u YOUR_USERNAME -p YOUR_GITHUB_TOKEN

# Push to registry
helm push n8n-1.95.3-4.tgz oci://ghcr.io/YOUR_USERNAME
```

### Making Chart Public

1. **Repository Visibility**: Make your GitHub repository public
2. **Package Visibility**:
   - Go to your GitHub profile → Packages
   - Find the `n8n` package
   - Go to Package settings → Change visibility → Public

## Development

### Testing Locally

```bash
# Lint the chart
helm lint .

# Template the chart
helm template n8n . --debug

# Install with dry-run
helm install n8n . --dry-run --debug
```

### Dependencies

This chart depends on:

- [PostgreSQL](https://github.com/bitnami/charts/tree/main/bitnami/postgresql) (Bitnami)
- [Redis](https://github.com/bitnami/charts/tree/main/bitnami/redis) (Bitnami)

## Security Considerations

⚠️ **Important**: Change default passwords and encryption keys in production!

- Set a strong `n8n.encryption_key`
- Configure strong passwords for PostgreSQL and Redis
- Use TLS/SSL for ingress
- Consider network policies for pod-to-pod communication

## Upgrading

```bash
# Update dependencies first
helm dependency update

# Upgrade the release
helm upgrade n8n . -f values.yaml
```

## Troubleshooting

### First-Time Setup Issues

If n8n shows a login screen instead of the setup wizard:

**Reset to Fresh Installation:**

```bash
# 1. Uninstall current release
helm uninstall n8n

# 2. Delete PostgreSQL PVC to reset database
kubectl delete pvc data-n8n-postgresql-0

# 3. Reinstall fresh
helm install n8n .
```

**Check Logs:**

```bash
kubectl logs deployment/n8n
kubectl logs statefulset/n8n-postgresql
```

### Common Issues

1. **Pods stuck in Pending**: Check PVC provisioning and resource limits
2. **Database connection errors**: Verify PostgreSQL is running and accessible
3. **Redis connection errors**: Check Redis master service availability

### Getting Setup Wizard

On first installation, n8n should show a setup wizard. If you see a login screen:

- Database may have existing data
- Use the reset procedure above
- Check logs for specific errors

## Uninstalling

```bash
helm uninstall n8n
```

**Complete Cleanup (includes data):**

```bash
# Uninstall release
helm uninstall n8n

# Delete all persistent data
kubectl delete pvc -l app.kubernetes.io/instance=n8n
```

**⚠️ Warning**: Deleting PVCs will permanently remove all n8n data, workflows, and database content.

## Support

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community](https://community.n8n.io/)
- [Helm Documentation](https://helm.sh/docs/)

## License

This chart is licensed under the MIT License. See the n8n project for its licensing terms.
