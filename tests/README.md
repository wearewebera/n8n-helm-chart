# N8N Helm Chart Unit Tests

This directory contains unit tests for the N8N Helm chart using [helm-unittest](https://github.com/helm-unittest/helm-unittest).

## Prerequisites

### Installing helm-unittest

```bash
# Install as Helm plugin
helm plugin install https://github.com/helm-unittest/helm-unittest.git

# Verify installation
helm plugin list
```

### Using Docker (alternative)

```bash
# Run tests with Docker
docker run -ti --rm -v $(pwd):/apps helmunittest/helm-unittest:latest .
```

## Test Structure

```
tests/
├── deployment_test.yaml      # Tests for Deployment
├── service_test.yaml         # Tests for Service
├── serviceaccount_test.yaml  # Tests for ServiceAccount
├── ingress_test.yaml         # Tests for Ingress
├── hpa_test.yaml            # Tests for HPA
├── pvc_test.yaml            # Tests for PVC
├── secret_test.yaml         # Tests for Secret
└── README.md                # This file
```

## Running Tests

### Run all tests

```bash
# From the chart root directory
helm unittest .

# With colored output
helm unittest --color .

# With verbose output
helm unittest -v .
```

### Run specific tests

```bash
# Only deployment tests
helm unittest -f 'tests/deployment_test.yaml' .

# Only service tests
helm unittest -f 'tests/service_test.yaml' .
```

### Useful options

```bash
# Stop on first error
helm unittest --failfast .

# Generate JUnit report (useful for CI/CD)
helm unittest -t junit -o test-results.xml .

# Update snapshots (when using snapshot testing)
helm unittest -u .

# Run with specific values
helm unittest -v values-production.yaml .
```

## Test Cases Covered

### Deployment (`deployment_test.yaml`)

- ✅ Rendering with default values
- ✅ Custom image tag configuration
- ✅ Replica configuration
- ✅ Replica disabling when autoscaling is enabled
- ✅ Image pull policy
- ✅ Security context
- ✅ Custom resources

### Service (`service_test.yaml`)

- ✅ Rendering with default values
- ✅ Custom service type
- ✅ Custom port
- ✅ Correct labels

### ServiceAccount (`serviceaccount_test.yaml`)

- ✅ Creation conditioned by `create` value
- ✅ Custom name
- ✅ Custom annotations
- ✅ Automount configuration

### Ingress (`ingress_test.yaml`)

- ✅ Rendering conditioned by `enabled` value
- ✅ Basic host and path configuration
- ✅ Ingress class
- ✅ Custom annotations
- ✅ TLS configuration
- ✅ Different Kubernetes API versions

### HPA (`hpa_test.yaml`)

- ✅ Rendering conditioned by autoscaling
- ✅ Min/max replica configuration
- ✅ CPU metrics
- ✅ Memory metrics
- ✅ Multiple metrics

### PVC (`pvc_test.yaml`)

- ✅ Rendering conditioned by persistence
- ✅ Storage size
- ✅ Access mode
- ✅ Storage class

### Secret (`secret_test.yaml`)

- ✅ Basic rendering
- ✅ Custom encryption key
- ✅ Automatic key generation
- ✅ Correct labels

## Adding New Tests

To add new tests:

1. Create a new `*_test.yaml` file in the `tests/` directory
2. Use the JSON schema for validation:
   ```yaml
   # yaml-language-server: $schema=https://raw.githubusercontent.com/helm-unittest/helm-unittest/main/schema/helm-testsuite.json
   ```
3. Define the basic structure:
   ```yaml
   suite: test <component-name>
   templates:
     - <template-file>.yaml
   tests:
     - it: should <test-description>
       asserts:
         - <assertion>
   ```

## CI/CD Integration

### GitHub Actions

```yaml
name: Helm Unit Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install Helm
        uses: azure/setup-helm@v3
      - name: Install helm-unittest
        run: helm plugin install https://github.com/helm-unittest/helm-unittest.git
      - name: Run tests
        run: helm unittest --color .
```

### GitLab CI

```yaml
helm-unittest:
  image: helmunittest/helm-unittest:latest
  script:
    - helm unittest --color .
  artifacts:
    reports:
      junit: test-results.xml
```

## Additional Resources

- [Official helm-unittest documentation](https://github.com/helm-unittest/helm-unittest/blob/main/DOCUMENT.md)
- [Community examples](https://github.com/helm-unittest/helm-unittest#open-source-community-examples)
- [JSON Schema for autocompletion](https://raw.githubusercontent.com/helm-unittest/helm-unittest/main/schema/helm-testsuite.json)
