# Project 5: Build a GitOps Pipeline with ArgoCD

## 🎯 Project Overview

**Difficulty:** Advanced  
**Duration:** 8-10 hours  
**Technologies:** ArgoCD, GitOps, Kubernetes, Helm, Kustomize

### What You'll Build
A complete GitOps pipeline using ArgoCD for automated Kubernetes deployments, with Git as the single source of truth, automated sync, rollback capabilities, and multi-environment management.

### Learning Objectives
- ✅ Understand GitOps principles
- ✅ Install and configure ArgoCD
- ✅ Implement declarative deployments
- ✅ Set up automated sync and self-healing
- ✅ Manage multiple environments
- ✅ Implement progressive delivery
- ✅ Configure RBAC and security
- ✅ Monitor deployment health

### Prerequisites
- Kubernetes cluster (Minikube or cloud)
- kubectl installed and configured
- Git repository (GitHub/GitLab)
- Helm installed
- Completed Projects 1-2 recommended

---

## 📋 Project Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Git Repository                           │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  manifests/                                             │    │
│  │  ├── base/                                              │    │
│  │  │   ├── deployment.yaml                               │    │
│  │  │   ├── service.yaml                                  │    │
│  │  │   └── kustomization.yaml                            │    │
│  │  ├── overlays/                                          │    │
│  │  │   ├── dev/                                           │    │
│  │  │   ├── staging/                                       │    │
│  │  │   └── production/                                    │    │
│  │  └── helm-charts/                                       │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                           │
                           │ Git Pull
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                            │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                  ArgoCD (Namespace: argocd)             │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │  ArgoCD Server                                    │  │    │
│  │  │  - UI Dashboard                                   │  │    │
│  │  │  - API Server                                     │  │    │
│  │  │  - Authentication                                 │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │  Application Controller                           │  │    │
│  │  │  - Sync Applications                              │  │    │
│  │  │  - Health Monitoring                              │  │    │
│  │  │  - Auto-sync & Self-healing                       │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │  Repo Server                                      │  │    │
│  │  │  - Git Repository Access                          │  │    │
│  │  │  - Manifest Generation                            │  │    │
│  │  │  - Helm/Kustomize Processing                      │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────┘    │
│                           │                                      │
│                           │ Deploy                               │
│                           ▼                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │              Application Namespaces                     │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │    │
│  │  │     Dev      │  │   Staging    │  │  Production  │ │    │
│  │  │  Environment │  │  Environment │  │  Environment │ │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘ │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Step-by-Step Implementation

### Phase 1: Install ArgoCD

#### Step 1.1: Create ArgoCD Namespace

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# Verify installation
kubectl get pods -n argocd
kubectl get svc -n argocd
```

#### Step 1.2: Access ArgoCD UI

```bash
# Port forward ArgoCD server
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Access UI
open https://localhost:8080
# Username: admin
# Password: <from above command>

# Change password
argocd account update-password
```

#### Step 1.3: Install ArgoCD CLI

```bash
# macOS
brew install argocd

# Linux
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd

# Login via CLI
argocd login localhost:8080 --username admin --password <password> --insecure

# Verify
argocd version
```

---

### Phase 2: Create Git Repository Structure

#### Step 2.1: Initialize Git Repository

```bash
# Create repository
mkdir -p gitops-argocd-demo
cd gitops-argocd-demo

# Initialize Git
git init

# Create directory structure
mkdir -p {manifests/{base,overlays/{dev,staging,production}},helm-charts,argocd-apps}
```

#### Step 2.2: Create Base Manifests

**File: `manifests/base/deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
```

**File: `manifests/base/service.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  labels:
    app: myapp
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
```

**File: `manifests/base/kustomization.yaml`**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml

commonLabels:
  app: myapp
  managed-by: argocd
```

#### Step 2.3: Create Environment Overlays

**File: `manifests/overlays/dev/kustomization.yaml`**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: dev

bases:
  - ../../base

namePrefix: dev-

replicas:
  - name: myapp
    count: 1

images:
  - name: nginx
    newTag: "1.21"

commonLabels:
  environment: dev

configMapGenerator:
  - name: app-config
    literals:
      - ENV=development
      - LOG_LEVEL=debug
```

**File: `manifests/overlays/staging/kustomization.yaml`**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: staging

bases:
  - ../../base

namePrefix: staging-

replicas:
  - name: myapp
    count: 2

images:
  - name: nginx
    newTag: "1.21"

commonLabels:
  environment: staging

configMapGenerator:
  - name: app-config
    literals:
      - ENV=staging
      - LOG_LEVEL=info
```

**File: `manifests/overlays/production/kustomization.yaml`**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production

bases:
  - ../../base

namePrefix: prod-

replicas:
  - name: myapp
    count: 3

images:
  - name: nginx
    newTag: "1.21"

commonLabels:
  environment: production

configMapGenerator:
  - name: app-config
    literals:
      - ENV=production
      - LOG_LEVEL=warn
```

---

### Phase 3: Create ArgoCD Applications

#### Step 3.1: Development Application

**File: `argocd-apps/dev-app.yaml`**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  
  source:
    repoURL: https://github.com/yourusername/gitops-argocd-demo.git
    targetRevision: main
    path: manifests/overlays/dev
  
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  
  revisionHistoryLimit: 10
```

#### Step 3.2: Staging Application

**File: `argocd-apps/staging-app.yaml`**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-staging
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  
  source:
    repoURL: https://github.com/yourusername/gitops-argocd-demo.git
    targetRevision: main
    path: manifests/overlays/staging
  
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
  
  revisionHistoryLimit: 10
```

#### Step 3.3: Production Application (Manual Sync)

**File: `argocd-apps/production-app.yaml`**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  annotations:
    notifications.argoproj.io/subscribe.on-sync-succeeded.slack: production-alerts
spec:
  project: default
  
  source:
    repoURL: https://github.com/yourusername/gitops-argocd-demo.git
    targetRevision: main
    path: manifests/overlays/production
  
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  
  syncPolicy:
    # Manual sync for production
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  
  revisionHistoryLimit: 10
```

#### Step 3.4: App of Apps Pattern

**File: `argocd-apps/app-of-apps.yaml`**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app-of-apps
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/yourusername/gitops-argocd-demo.git
    targetRevision: main
    path: argocd-apps
  
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

### Phase 4: Deploy Applications

#### Step 4.1: Push to Git

```bash
# Add all files
git add .

# Commit
git commit -m "Initial GitOps setup with ArgoCD"

# Add remote (replace with your repo)
git remote add origin https://github.com/yourusername/gitops-argocd-demo.git

# Push
git push -u origin main
```

#### Step 4.2: Deploy via ArgoCD CLI

```bash
# Create dev application
argocd app create myapp-dev \
  --repo https://github.com/yourusername/gitops-argocd-demo.git \
  --path manifests/overlays/dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace dev \
  --sync-policy automated \
  --auto-prune \
  --self-heal

# Create staging application
argocd app create myapp-staging \
  --repo https://github.com/yourusername/gitops-argocd-demo.git \
  --path manifests/overlays/staging \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace staging \
  --sync-policy automated

# Create production application (manual sync)
argocd app create myapp-production \
  --repo https://github.com/yourusername/gitops-argocd-demo.git \
  --path manifests/overlays/production \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production

# Or apply via kubectl
kubectl apply -f argocd-apps/dev-app.yaml
kubectl apply -f argocd-apps/staging-app.yaml
kubectl apply -f argocd-apps/production-app.yaml
```

#### Step 4.3: Sync Applications

```bash
# Sync dev application
argocd app sync myapp-dev

# Watch sync status
argocd app wait myapp-dev --health

# Get application details
argocd app get myapp-dev

# List all applications
argocd app list
```

---

### Phase 5: GitOps Workflows

#### Step 5.1: Update Application

```bash
# Update image version in Git
cd manifests/overlays/dev
sed -i 's/newTag: "1.21"/newTag: "1.22"/' kustomization.yaml

# Commit and push
git add .
git commit -m "Update nginx to 1.22 in dev"
git push

# ArgoCD will automatically sync (if auto-sync enabled)
# Or manually sync
argocd app sync myapp-dev
```

#### Step 5.2: Rollback

```bash
# View history
argocd app history myapp-dev

# Rollback to previous version
argocd app rollback myapp-dev <revision-id>

# Or via Git
git revert HEAD
git push
```

#### Step 5.3: Progressive Delivery

**File: `argocd-apps/canary-rollout.yaml`**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-rollout
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {duration: 1m}
      - setWeight: 40
      - pause: {duration: 1m}
      - setWeight: 60
      - pause: {duration: 1m}
      - setWeight: 80
      - pause: {duration: 1m}
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.21
```

---

## 🧪 Testing and Validation

### Test 1: Verify Sync

```bash
# Check application status
argocd app get myapp-dev

# Check sync status
kubectl get applications -n argocd

# Verify resources
kubectl get all -n dev
kubectl get all -n staging
kubectl get all -n production
```

### Test 2: Test Self-Healing

```bash
# Delete a pod manually
kubectl delete pod -n dev -l app=myapp

# ArgoCD will automatically recreate it
kubectl get pods -n dev -w
```

### Test 3: Test Auto-Sync

```bash
# Make a change in Git
echo "# Test change" >> manifests/base/deployment.yaml
git add .
git commit -m "Test auto-sync"
git push

# Watch ArgoCD sync
argocd app wait myapp-dev --sync
```

---

## 📊 Monitoring and Observability

### ArgoCD Metrics

```bash
# Port forward Prometheus
kubectl port-forward -n argocd svc/argocd-metrics 8082:8082

# Access metrics
curl http://localhost:8082/metrics

# Key metrics:
# - argocd_app_sync_total
# - argocd_app_health_status
# - argocd_app_sync_status
```

### ArgoCD Notifications

**File: `argocd-notifications-cm.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.slack: |
    token: $slack-token
  
  template.app-deployed: |
    message: |
      Application {{.app.metadata.name}} is now running new version.
    slack:
      attachments: |
        [{
          "title": "{{ .app.metadata.name}}",
          "title_link":"{{.context.argocdUrl}}/applications/{{.app.metadata.name}}",
          "color": "#18be52",
          "fields": [
          {
            "title": "Sync Status",
            "value": "{{.app.status.sync.status}}",
            "short": true
          },
          {
            "title": "Repository",
            "value": "{{.app.spec.source.repoURL}}",
            "short": true
          }
          ]
        }]
  
  trigger.on-deployed: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-deployed]
```

---

## 🎯 Success Criteria

- [ ] ArgoCD installed and accessible
- [ ] Applications deployed via GitOps
- [ ] Auto-sync working
- [ ] Self-healing functional
- [ ] Multi-environment setup working
- [ ] Rollback capability tested
- [ ] Notifications configured
- [ ] RBAC implemented

---

## 📚 Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [GitOps Principles](https://www.gitops.tech/)
- [Kustomize Documentation](https://kustomize.io/)
- [ArgoCD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)

---

## 🎓 What You Learned

✅ GitOps principles and practices  
✅ ArgoCD installation and configuration  
✅ Declarative application deployment  
✅ Multi-environment management  
✅ Automated sync and self-healing  
✅ Progressive delivery strategies  
✅ RBAC and security  
✅ Monitoring and notifications  

---

## 🚀 Next Steps

1. **Project 6**: Secure CI/CD with Trivy + Vault
2. Implement ArgoCD Image Updater
3. Add Argo Rollouts for advanced deployments
4. Integrate with CI/CD pipeline
5. Implement multi-cluster management

---

**Project Status:** ✅ Complete  
**Last Updated:** 2024-01-16