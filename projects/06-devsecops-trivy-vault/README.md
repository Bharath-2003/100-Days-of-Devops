# Project 6: Secure a CI/CD Pipeline (Trivy + Vault)

## 🎯 Project Overview

**Difficulty:** Advanced  
**Duration:** 8-10 hours  
**Technologies:** Trivy, HashiCorp Vault, GitHub Actions, Docker, Kubernetes

### What You'll Build
A secure CI/CD pipeline with automated security scanning using Trivy for vulnerabilities and HashiCorp Vault for secrets management, implementing DevSecOps best practices.

### Learning Objectives
- ✅ Implement security scanning in CI/CD
- ✅ Set up HashiCorp Vault
- ✅ Manage secrets securely
- ✅ Scan container images for vulnerabilities
- ✅ Implement security gates
- ✅ Configure SAST and DAST
- ✅ Automate security compliance
- ✅ Integrate security into DevOps workflow

### Prerequisites
- Docker installed
- Kubernetes cluster (optional)
- GitHub account
- Basic understanding of security concepts
- Completed Projects 1-2 recommended

---

## 📋 Project Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      CI/CD Pipeline                              │
│                                                                   │
│  ┌────────────────────────────────────────────────────────┐    │
│  │              GitHub Actions Workflow                    │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │  1. Code Checkout                                 │  │    │
│  │  │  2. Dependency Scanning (Trivy)                   │  │    │
│  │  │  3. SAST (Static Analysis)                        │  │    │
│  │  │  4. Build Docker Image                            │  │    │
│  │  │  5. Image Scanning (Trivy)                        │  │    │
│  │  │  6. Security Gate (Pass/Fail)                     │  │    │
│  │  │  7. Push to Registry (if passed)                  │  │    │
│  │  │  8. Deploy (with Vault secrets)                   │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────┘    │
│                           │                                      │
│                           │ Scan Results                         │
│                           ▼                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                  Trivy Scanner                          │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │  - Vulnerability Database                        │  │    │
│  │  │  - CVE Detection                                 │  │    │
│  │  │  - Severity Classification                       │  │    │
│  │  │  - SBOM Generation                               │  │    │
│  │  │  - Policy Enforcement                            │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────┘    │
│                           │                                      │
│                           │ Fetch Secrets                        │
│                           ▼                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │              HashiCorp Vault                            │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │  - Secret Storage                                │  │    │
│  │  │  - Dynamic Secrets                               │  │    │
│  │  │  - Encryption as a Service                       │  │    │
│  │  │  - Access Control (Policies)                     │  │    │
│  │  │  - Audit Logging                                 │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Step-by-Step Implementation

### Phase 1: Set Up HashiCorp Vault

#### Step 1.1: Install Vault

```bash
# macOS
brew install vault

# Linux
wget https://releases.hashicorp.com/vault/1.15.0/vault_1.15.0_linux_amd64.zip
unzip vault_1.15.0_linux_amd64.zip
sudo mv vault /usr/local/bin/

# Verify installation
vault version
```

#### Step 1.2: Start Vault Server (Dev Mode)

```bash
# Start Vault in dev mode (for testing)
vault server -dev

# In another terminal, set environment variables
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='<root-token-from-output>'

# Check status
vault status
```

#### Step 1.3: Initialize Vault (Production Mode)

**File: `vault/config.hcl`**
```hcl
storage "file" {
  path = "./vault/data"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1
}

api_addr = "http://127.0.0.1:8200"
cluster_addr = "https://127.0.0.1:8201"
ui = true
```

```bash
# Start Vault with config
vault server -config=vault/config.hcl

# Initialize Vault
vault operator init

# Unseal Vault (use 3 of 5 keys)
vault operator unseal <key1>
vault operator unseal <key2>
vault operator unseal <key3>

# Login with root token
vault login <root-token>
```

#### Step 1.4: Configure Vault Secrets

```bash
# Enable KV secrets engine
vault secrets enable -path=secret kv-v2

# Store secrets
vault kv put secret/myapp/config \
  db_password="supersecret123" \
  api_key="abc123xyz789" \
  jwt_secret="my-jwt-secret"

# Read secrets
vault kv get secret/myapp/config

# Enable AppRole auth
vault auth enable approle

# Create policy
vault policy write myapp-policy - <<EOF
path "secret/data/myapp/*" {
  capabilities = ["read", "list"]
}
EOF

# Create AppRole
vault write auth/approle/role/myapp \
  token_policies="myapp-policy" \
  token_ttl=1h \
  token_max_ttl=4h

# Get Role ID
vault read auth/approle/role/myapp/role-id

# Generate Secret ID
vault write -f auth/approle/role/myapp/secret-id
```

---

### Phase 2: Set Up Trivy

#### Step 2.1: Install Trivy

```bash
# macOS
brew install trivy

# Linux
wget https://github.com/aquasecurity/trivy/releases/download/v0.48.0/trivy_0.48.0_Linux-64bit.tar.gz
tar zxvf trivy_0.48.0_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/

# Verify installation
trivy version
```

#### Step 2.2: Scan Docker Image

```bash
# Scan image
trivy image nginx:latest

# Scan with severity filter
trivy image --severity HIGH,CRITICAL nginx:latest

# Generate JSON report
trivy image --format json --output results.json nginx:latest

# Scan filesystem
trivy fs .

# Scan Kubernetes manifests
trivy config ./k8s-manifests/
```

#### Step 2.3: Create Trivy Configuration

**File: `.trivyignore`**
```
# Ignore specific CVEs
CVE-2021-12345
CVE-2021-67890

# Ignore by package
pkg:npm/lodash@4.17.20
```

**File: `trivy.yaml`**
```yaml
severity:
  - CRITICAL
  - HIGH

vulnerability:
  type:
    - os
    - library

scan:
  skip-dirs:
    - node_modules
    - vendor
  skip-files:
    - "*.test.js"

output:
  format: json
  template: "@contrib/html.tpl"
```

---

### Phase 3: Create Secure CI/CD Pipeline

#### Step 3.1: GitHub Actions Workflow

**File: `.github/workflows/secure-pipeline.yml`**
```yaml
name: Secure CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  IMAGE_NAME: myapp
  REGISTRY: ghcr.io

jobs:
  security-scan:
    name: Security Scanning
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Run Trivy vulnerability scanner (filesystem)
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-fs-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-fs-results.sarif'
      
      - name: Run Trivy vulnerability scanner (config)
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'config'
          scan-ref: '.'
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'

  sast-scan:
    name: Static Application Security Testing
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten

  build-and-scan:
    name: Build and Scan Docker Image
    runs-on: ubuntu-latest
    needs: [security-scan, sast-scan]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Build Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: false
          tags: ${{ env.IMAGE_NAME }}:${{ github.sha }}
          load: true
      
      - name: Run Trivy vulnerability scanner (image)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ env.IMAGE_NAME }}:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-image-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload Trivy image results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-image-results.sarif'
      
      - name: Security Gate
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ env.IMAGE_NAME }}:${{ github.sha }}'
          format: 'table'
          exit-code: '1'
          ignore-unfixed: true
          severity: 'CRITICAL'
      
      - name: Generate SBOM
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ env.IMAGE_NAME }}:${{ github.sha }}'
          format: 'cyclonedx'
          output: 'sbom.json'
      
      - name: Upload SBOM
        uses: actions/upload-artifact@v3
        with:
          name: sbom
          path: sbom.json

  deploy:
    name: Deploy with Vault Secrets
    runs-on: ubuntu-latest
    needs: build-and-scan
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Import Secrets from Vault
        uses: hashicorp/vault-action@v2
        with:
          url: ${{ secrets.VAULT_ADDR }}
          token: ${{ secrets.VAULT_TOKEN }}
          secrets: |
            secret/data/myapp/config db_password | DB_PASSWORD ;
            secret/data/myapp/config api_key | API_KEY ;
            secret/data/myapp/config jwt_secret | JWT_SECRET
      
      - name: Deploy Application
        run: |
          echo "Deploying with secrets from Vault"
          echo "DB_PASSWORD is set: ${DB_PASSWORD:+yes}"
          echo "API_KEY is set: ${API_KEY:+yes}"
          # Add your deployment commands here
```

#### Step 3.2: Docker Security Best Practices

**File: `Dockerfile`**
```dockerfile
# Use specific version, not latest
FROM node:18.19.0-alpine3.19 AS builder

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy application code
COPY --chown=nodejs:nodejs . .

# Production stage
FROM node:18.19.0-alpine3.19

# Install security updates
RUN apk update && \
    apk upgrade && \
    apk add --no-cache dumb-init && \
    rm -rf /var/cache/apk/*

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy from builder
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app .

# Switch to non-root user
USER nodejs

# Use dumb-init to handle signals
ENTRYPOINT ["dumb-init", "--"]

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js || exit 1

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "server.js"]
```

---

### Phase 4: Kubernetes Security

#### Step 4.1: Pod Security Policy

**File: `k8s/pod-security-policy.yaml`**
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: true
```

#### Step 4.2: Network Policy

**File: `k8s/network-policy.yaml`**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: myapp-network-policy
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 3000
  egress:
    - to:
        - podSelector:
            matchLabels:
              role: database
      ports:
        - protocol: TCP
          port: 5432
```

#### Step 4.3: Vault Injector

**File: `k8s/deployment-with-vault.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "myapp"
        vault.hashicorp.com/agent-inject-secret-config: "secret/data/myapp/config"
        vault.hashicorp.com/agent-inject-template-config: |
          {{- with secret "secret/data/myapp/config" -}}
          export DB_PASSWORD="{{ .Data.data.db_password }}"
          export API_KEY="{{ .Data.data.api_key }}"
          export JWT_SECRET="{{ .Data.data.jwt_secret }}"
          {{- end }}
    spec:
      serviceAccountName: myapp
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001
      containers:
      - name: myapp
        image: myapp:latest
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1001
          capabilities:
            drop:
              - ALL
        resources:
          limits:
            memory: "256Mi"
            cpu: "200m"
          requests:
            memory: "128Mi"
            cpu: "100m"
```

---

## 🧪 Testing and Validation

### Test 1: Scan Local Image

```bash
# Build image
docker build -t myapp:test .

# Scan with Trivy
trivy image myapp:test

# Scan with specific severity
trivy image --severity CRITICAL,HIGH myapp:test

# Generate report
trivy image --format json --output report.json myapp:test
```

### Test 2: Test Vault Integration

```bash
# Store secret
vault kv put secret/myapp/test password="test123"

# Retrieve secret
vault kv get secret/myapp/test

# Test AppRole auth
vault write auth/approle/login \
  role_id="<role-id>" \
  secret_id="<secret-id>"
```

### Test 3: Security Compliance

```bash
# Run CIS Docker Benchmark
docker run --rm --net host --pid host --userns host --cap-add audit_control \
  -v /var/lib:/var/lib \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /etc:/etc \
  aquasec/docker-bench-security

# Scan Kubernetes cluster
trivy k8s --report summary cluster
```

---

## 📊 Security Metrics

### Key Metrics to Track

```yaml
# Vulnerability metrics
- Total vulnerabilities found
- Critical vulnerabilities
- High severity vulnerabilities
- Time to remediate

# Compliance metrics
- Policy violations
- Security gate failures
- Scan coverage

# Secret management
- Secret rotation frequency
- Access audit logs
- Policy violations
```

---

## 🎯 Success Criteria

- [ ] Trivy scanning integrated in CI/CD
- [ ] Vault installed and configured
- [ ] Secrets managed via Vault
- [ ] Security gates implemented
- [ ] SBOM generated
- [ ] Container images scanned
- [ ] Kubernetes security policies applied
- [ ] Audit logging enabled

---

## 📚 Additional Resources

- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [HashiCorp Vault Documentation](https://www.vaultproject.io/docs)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)

---

## 🎓 What You Learned

✅ Security scanning with Trivy  
✅ Secrets management with Vault  
✅ DevSecOps practices  
✅ Container security  
✅ Kubernetes security  
✅ SBOM generation  
✅ Security gates in CI/CD  
✅ Compliance automation  

---

## 🚀 Next Steps

1. **Project 7**: Cloud Resume Challenge
2. Implement DAST scanning
3. Add runtime security (Falco)
4. Implement secret rotation
5. Add compliance scanning (OPA)

---

**Project Status:** ✅ Complete  
**Last Updated:** 2024-01-16