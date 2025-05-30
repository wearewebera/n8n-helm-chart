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
# Add the chart repository
helm install n8n oci://ghcr.io/wearewebera/n8n --version 0.1.0

# Or with custom values
helm install n8n oci://ghcr.io/wearewebera/n8n --version 0.1.0 -f custom-values.yaml
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

## Configuration

### Key Configuration Options

| Parameter                 | Description               | Default                        |
| ------------------------- | ------------------------- | ------------------------------ |
| `replicaCount`            | Number of n8n replicas    | `1`                            |
| `image.repository`        | n8n image repository      | `n8nio/n8n`                    |
| `image.tag`               | n8n image tag             | `1.69.0`                       |
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
   - Set tag version (e.g., `v0.2.0`)
   - Add release notes describing changes
   - Click "Publish release"
3. **GitHub Action**: The workflow automatically triggers and publishes to GitHub Container Registry

### Manual Publishing

```bash
# Package the chart
helm dependency update
helm package .

# Login to GitHub Container Registry
helm registry login ghcr.io -u YOUR_USERNAME -p YOUR_GITHUB_TOKEN

# Push to registry
helm push n8n-0.1.0.tgz oci://ghcr.io/YOUR_USERNAME
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
