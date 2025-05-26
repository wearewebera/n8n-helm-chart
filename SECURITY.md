# Security Configuration

## ✅ **Security Features Implemented**

### **1. Secure Secrets Management**
- **Auto-generated passwords**: PostgreSQL and Redis passwords are automatically generated if not provided
- **Encryption key management**: Secure random 32-character encryption key generation
- **External secrets support**: Option to use existing Kubernetes secrets

### **2. Restrictive Security Contexts**
- **Non-root execution**: All containers run as non-root user (UID 1000)
- **Read-only root filesystem**: Prevents runtime modifications
- **Dropped capabilities**: All Linux capabilities dropped
- **Seccomp profiles**: Runtime security profiles enabled

### **3. Network Security**
- **Pod anti-affinity**: Prevents pods from being scheduled on same node
- **Conditional environment variables**: Only set when services are enabled

## 🔒 **Production Security Checklist**

### **Required Actions for Production**

1. **Set Encryption Key**:
   ```yaml
   n8n:
     encryption_key: "your-secure-32-character-key"
   ```
   Generate with: `openssl rand -hex 16`

2. **Configure Database Passwords**:
   ```yaml
   postgresql:
     auth:
       postgresPassword: "your-secure-postgres-password"
       password: "your-secure-user-password"
   
   redis:
     auth:
       password: "your-secure-redis-password"
   ```

3. **Use External Secrets** (Recommended):
   ```yaml
   n8n:
     existingSecret: "n8n-secrets"
     existingSecretKey: "encryption-key"
   
   postgresql:
     auth:
       existingSecret: "postgres-secrets"
   
   redis:
     auth:
       existingSecret: "redis-secrets"
   ```

### **Optional Enhancements**

4. **Network Policies** (if supported):
   ```yaml
   # Add to your cluster
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: n8n-network-policy
   spec:
     podSelector:
       matchLabels:
         app.kubernetes.io/name: n8n
     ingress:
     - from:
       - podSelector:
           matchLabels:
             app.kubernetes.io/name: ingress-nginx
   ```

5. **Pod Security Standards**:
   ```yaml
   # Add to namespace
   apiVersion: v1
   kind: Namespace
   metadata:
     name: n8n
     labels:
       pod-security.kubernetes.io/enforce: restricted
       pod-security.kubernetes.io/audit: restricted
       pod-security.kubernetes.io/warn: restricted
   ```

## 🚨 **Security Warnings**

### **Development vs Production**

**Development (Local)**:
- ✅ Auto-generated passwords are acceptable
- ✅ Default encryption key rotation not critical
- ✅ Single node deployment is fine

**Production**:
- ❌ NEVER use auto-generated passwords
- ❌ NEVER use default encryption keys
- ❌ ALWAYS use external secret management
- ❌ ALWAYS enable TLS/HTTPS
- ❌ ALWAYS implement backup strategies

### **Critical Security Requirements**

1. **Encryption Key**: Must be unique per environment and securely stored
2. **Database Passwords**: Must be complex and rotated regularly
3. **TLS Termination**: Enable HTTPS at ingress level
4. **Backup Encryption**: Encrypt all backups and store securely
5. **Access Control**: Implement RBAC and network policies

## 🔧 **Validation Commands**

```bash
# Check security contexts
kubectl get pod -o jsonpath='{.spec.securityContext}' <pod-name>

# Verify read-only filesystem
kubectl exec <pod-name> -- touch /test 2>&1 | grep "Read-only"

# Check running user
kubectl exec <pod-name> -- id

# Verify secret references
kubectl describe deployment <deployment-name> | grep -A5 "Environment"
```

## 📋 **Security Compliance**

This chart implements security controls aligned with:
- **CIS Kubernetes Benchmark** recommendations
- **NIST Cybersecurity Framework** guidelines  
- **OWASP Container Security** best practices
- **Kubernetes Pod Security Standards** (Restricted)

For additional security hardening, consider implementing:
- Service mesh (Istio/Linkerd) for mTLS
- Policy engines (OPA Gatekeeper/Kyverno)
- Runtime security (Falco/Sysdig)
- Vulnerability scanning (Trivy/Twistlock)