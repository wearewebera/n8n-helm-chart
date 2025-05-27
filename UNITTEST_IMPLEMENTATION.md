# N8N Helm Chart - Unit Testing Support

## 📋 Implementation Summary

Successfully implemented complete unit testing support for your N8N Helm chart using the **helm-unittest** tool. The implementation includes:

## 🗂️ Created File Structure

```
n8n-helm-chart/
├── .helmignore                     # Updated to exclude tests and snapshots
├── .github/
│   ├── workflows/test.yml          # CI/CD with GitHub Actions
│   └── ct.yaml                     # chart-testing configuration
├── .vscode/
│   └── settings.json               # VS Code settings for YAML
├── tests/                          # 📂 Main test directory
│   ├── deployment_test.yaml        # Deployment tests
│   ├── service_test.yaml           # Service tests
│   ├── serviceaccount_test.yaml    # ServiceAccount tests
│   ├── ingress_test.yaml           # Ingress tests
│   ├── hpa_test.yaml               # HPA tests
│   ├── pvc_test.yaml               # PVC tests
│   ├── secret_test.yaml            # Secret tests
│   ├── values-test.yaml            # Test values
│   └── README.md                   # Detailed test documentation
├── scripts/
│   └── setup-dev.sh               # Automatic setup script
├── Makefile                        # Facilitated development commands
└── README.md                       # Updated with testing section
```

## 🧪 Implemented Test Coverage

### ✅ Deployment (`deployment_test.yaml`)

- Rendering with default values
- Custom image tag configuration
- Replica configuration
- Replica disabling when autoscaling is enabled
- Image pull policy
- Security context
- Custom resources (CPU/Memory)

### ✅ Service (`service_test.yaml`)

- Rendering with default values
- Custom service type (ClusterIP, NodePort, LoadBalancer)
- Custom port configuration
- Correct labels and selectors

### ✅ ServiceAccount (`serviceaccount_test.yaml`)

- Conditional creation based on `create` value
- Custom ServiceAccount name
- Custom annotations
- automountServiceAccountToken configuration

### ✅ Ingress (`ingress_test.yaml`)

- Conditional rendering based on `enabled` value
- Basic host and path configuration
- Ingress class (ingressClassName)
- Custom annotations
- TLS configuration
- Support for different Kubernetes API versions

### ✅ HPA (`hpa_test.yaml`)

- Conditional rendering based on autoscaling
- Min/max replica configuration
- CPU metrics (targetCPUUtilizationPercentage)
- Memory metrics (targetMemoryUtilizationPercentage)
- Multiple metrics simultaneously

### ✅ PVC (`pvc_test.yaml`)

- Conditional rendering based on persistence
- Custom storage size
- Access mode (ReadWriteOnce, ReadWriteMany)
- Custom storage class

### ✅ Secret (`secret_test.yaml`)

- Basic Secret rendering
- Custom encryption key
- Automatic random key generation
- Correct labels

## 🚀 How to Use

### 1. Install helm-unittest

```bash
# Method 1: Helm Plugin (recommended)
helm plugin install https://github.com/helm-unittest/helm-unittest.git

# Method 2: Docker (alternative without installation)
docker run -ti --rm -v $(pwd):/apps helmunittest/helm-unittest:latest .
```

### 2. Run Tests

```bash
# Run all tests
helm unittest .

# Run with colored and verbose output
helm unittest --color -v .

# Run specific test
helm unittest -f 'tests/deployment_test.yaml' .

# Generate JUnit report (for CI/CD)
helm unittest -t junit -o test-results.xml .
```

### 3. Using the Makefile

```bash
# See available commands
make help

# Setup development environment
make dev-setup

# Run all tests
make test

# Run specific tests
make test-deployment
make test-service
make test-ingress

# Run tests with JUnit
make test-junit

# Validate (lint + test)
make validate
```

### 4. Automatic Setup Script

```bash
# Run setup script
./scripts/setup-dev.sh
```

## 🔧 CI/CD Integration

### GitHub Actions

- **Automatic** Helm chart linting
- **Unit tests** with helm-unittest
- **Integration tests** with chart-testing
- **Security scanning** with Trivy
- **Test reports** in JUnit format

### CI/CD Commands

```bash
# For use in pipelines
helm unittest --color --output-type junit --output-file test-results.xml .
```

## 📚 Additional Resources

### Documentation

- `tests/README.md` - Detailed test guide
- `Makefile` - Facilitated commands
- `.vscode/settings.json` - YAML autocompletion with schema

### JSON Schema

All test files include the JSON schema for validation and autocompletion:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/helm-unittest/helm-unittest/main/schema/helm-testsuite.json
```

### Usage Examples

- `tests/values-test.yaml` - Custom values for tests
- Tests cover real production scenarios
- Validation of different configurations

## 🎯 Next Steps

1. **Install helm-unittest**: `helm plugin install https://github.com/helm-unittest/helm-unittest.git`
2. **Run tests**: `helm unittest --color .`
3. **Review coverage**: Check if all use cases are covered
4. **Setup CI/CD**: Use the provided GitHub Actions workflow
5. **Extend tests**: Add tests for new features as needed

## 📖 References

- [helm-unittest GitHub](https://github.com/helm-unittest/helm-unittest)
- [Official documentation](https://github.com/helm-unittest/helm-unittest/blob/main/DOCUMENT.md)
- [Community examples](https://github.com/helm-unittest/helm-unittest#open-source-community-examples)

---

**✨ Implementation completed successfully!**

Your N8N Helm chart now has a complete unit test suite that ensures quality and reliability of the chart in different configuration scenarios.
