# Project 2: Deploy App on Kubernetes (Local with Minikube)

## 🎯 Project Overview

**Difficulty:** Intermediate  
**Duration:** 6-8 hours  
**Technologies:** Kubernetes, kubectl, Helm, Minikube, Docker

### What You'll Build
A complete Kubernetes deployment running locally on Minikube, including deployments, services, ingress, ConfigMaps, Secrets, and Helm charts.

### Learning Objectives
- ✅ Set up local Kubernetes cluster with Minikube
- ✅ Create Kubernetes manifests (Deployments, Services, ConfigMaps, Secrets)
- ✅ Implement Ingress for external access
- ✅ Use Helm for package management
- ✅ Implement rolling updates and rollbacks
- ✅ Set up monitoring and logging
- ✅ Implement autoscaling

### Prerequisites
- Completed Project 1 (Docker + GitHub Actions)
- Docker installed and running
- Basic understanding of YAML
- 8GB RAM minimum for Minikube

---

## 📋 Project Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Minikube Cluster                          │
│  ┌────────────────────────────────────────────────────┐    │
│  │                   Ingress Controller                │    │
│  │              (nginx-ingress)                        │    │
│  └────────────────────────────────────────────────────┘    │
│                           │                                  │
│                           ▼                                  │
│  ┌────────────────────────────────────────────────────┐    │
│  │                    Service (ClusterIP)              │    │
│  │                  myapp-service:80                   │    │
│  └────────────────────────────────────────────────────┘    │
│                           │                                  │
│                           ▼                                  │
│  ┌────────────────────────────────────────────────────┐    │
│  │              Deployment (3 replicas)                │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │    │
│  │  │  Pod 1   │  │  Pod 2   │  │  Pod 3   │        │    │
│  │  │ myapp:v1 │  │ myapp:v1 │  │ myapp:v1 │        │    │
│  │  └──────────┘  └──────────┘  └──────────┘        │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │              ConfigMap & Secrets                    │    │
│  │  - Environment variables                            │    │
│  │  - Configuration files                              │    │
│  │  - Sensitive data                                   │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │         Persistent Volume (Optional)                │    │
│  │              hostPath storage                       │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Step-by-Step Implementation

### Phase 1: Set Up Minikube

#### Step 1.1: Install Minikube

**macOS:**
```bash
# Using Homebrew
brew install minikube

# Or download binary
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

**Linux:**
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**Windows:**
```powershell
# Using Chocolatey
choco install minikube

# Or download installer from https://minikube.sigs.k8s.io/docs/start/
```

#### Step 1.2: Install kubectl

```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Windows
choco install kubernetes-cli

# Verify installation
kubectl version --client
```

#### Step 1.3: Start Minikube

```bash
# Start Minikube with specific resources
minikube start --cpus=4 --memory=8192 --driver=docker

# Verify cluster is running
minikube status

# Enable addons
minikube addons enable ingress
minikube addons enable metrics-server
minikube addons enable dashboard

# View dashboard
minikube dashboard
```

---

### Phase 2: Create Kubernetes Manifests

#### Step 2.1: Create Project Structure

```bash
mkdir -p k8s-minikube-demo/{manifests,helm-chart,scripts}
cd k8s-minikube-demo

# Create directory structure
mkdir -p manifests/{base,overlays/{dev,prod}}
mkdir -p helm-chart/myapp/{templates,charts}
```

#### Step 2.2: Create Namespace

**File: `manifests/base/namespace.yaml`**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
  labels:
    name: myapp
    environment: development
```

#### Step 2.3: Create ConfigMap

**File: `manifests/base/configmap.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: myapp
data:
  APP_NAME: "MyApp"
  APP_VERSION: "1.0.0"
  LOG_LEVEL: "info"
  NODE_ENV: "production"
  # Application configuration
  app.properties: |
    server.port=3000
    server.host=0.0.0.0
    logging.level=info
```

#### Step 2.4: Create Secret

**File: `manifests/base/secret.yaml`**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
  namespace: myapp
type: Opaque
data:
  # Base64 encoded values
  # echo -n 'mysecretpassword' | base64
  DB_PASSWORD: bXlzZWNyZXRwYXNzd29yZA==
  API_KEY: YXBpa2V5MTIzNDU2Nzg5MA==
stringData:
  # Plain text (will be encoded automatically)
  JWT_SECRET: "my-jwt-secret-key"
```

#### Step 2.5: Create Deployment

**File: `manifests/base/deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  namespace: myapp
  labels:
    app: myapp
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: yourusername/myapp:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
          name: http
          protocol: TCP
        env:
        - name: PORT
          value: "3000"
        - name: NODE_ENV
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: NODE_ENV
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: LOG_LEVEL
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: DB_PASSWORD
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: API_KEY
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
          readOnly: true
      volumes:
      - name: config-volume
        configMap:
          name: myapp-config
          items:
          - key: app.properties
            path: app.properties
```

#### Step 2.6: Create Service

**File: `manifests/base/service.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: myapp
  labels:
    app: myapp
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
    name: http
  sessionAffinity: ClientIP
```

#### Step 2.7: Create Ingress

**File: `manifests/base/ingress.yaml`**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: myapp
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

#### Step 2.8: Create HorizontalPodAutoscaler

**File: `manifests/base/hpa.yaml`**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: myapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
      - type: Pods
        value: 2
        periodSeconds: 30
      selectPolicy: Max
```

---

### Phase 3: Deploy to Minikube

#### Step 3.1: Apply Manifests

```bash
# Create namespace
kubectl apply -f manifests/base/namespace.yaml

# Apply all manifests
kubectl apply -f manifests/base/

# Verify deployment
kubectl get all -n myapp

# Check pods
kubectl get pods -n myapp -w

# Check services
kubectl get svc -n myapp

# Check ingress
kubectl get ingress -n myapp
```

#### Step 3.2: Configure Local DNS

```bash
# Get Minikube IP
minikube ip

# Add to /etc/hosts (Linux/macOS)
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts

# Windows: Add to C:\Windows\System32\drivers\etc\hosts
# <minikube-ip> myapp.local
```

#### Step 3.3: Test Application

```bash
# Test via curl
curl http://myapp.local

# Test health endpoint
curl http://myapp.local/health

# Test API endpoint
curl http://myapp.local/api/users

# Port forward (alternative)
kubectl port-forward -n myapp svc/myapp-service 8080:80

# Test via port forward
curl http://localhost:8080
```

---

### Phase 4: Create Helm Chart

#### Step 4.1: Initialize Helm Chart

```bash
cd helm-chart
helm create myapp

# Or manually create structure
mkdir -p myapp/{templates,charts}
```

#### Step 4.2: Create Chart.yaml

**File: `helm-chart/myapp/Chart.yaml`**
```yaml
apiVersion: v2
name: myapp
description: A Helm chart for MyApp Kubernetes deployment
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - nodejs
  - web
  - api
maintainers:
  - name: Your Name
    email: your.email@example.com
```

#### Step 4.3: Create values.yaml

**File: `helm-chart/myapp/values.yaml`**
```yaml
replicaCount: 3

image:
  repository: yourusername/myapp
  pullPolicy: Always
  tag: "latest"

nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 80
  targetPort: 3000

ingress:
  enabled: true
  className: "nginx"
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  hosts:
    - host: myapp.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

config:
  appName: "MyApp"
  appVersion: "1.0.0"
  logLevel: "info"
  nodeEnv: "production"

secrets:
  dbPassword: "mysecretpassword"
  apiKey: "apikey1234567890"
  jwtSecret: "my-jwt-secret-key"
```

#### Step 4.4: Create Deployment Template

**File: `helm-chart/myapp/templates/deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
          protocol: TCP
        env:
        - name: NODE_ENV
          value: {{ .Values.config.nodeEnv }}
        - name: LOG_LEVEL
          value: {{ .Values.config.logLevel }}
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ include "myapp.fullname" . }}-secret
              key: dbPassword
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 10
          periodSeconds: 5
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

#### Step 4.5: Deploy with Helm

```bash
# Install chart
helm install myapp ./helm-chart/myapp -n myapp --create-namespace

# Upgrade release
helm upgrade myapp ./helm-chart/myapp -n myapp

# List releases
helm list -n myapp

# Get values
helm get values myapp -n myapp

# Uninstall
helm uninstall myapp -n myapp
```

---

## 🧪 Testing and Validation

### Test 1: Rolling Update

```bash
# Update image version
kubectl set image deployment/myapp-deployment myapp=yourusername/myapp:v2 -n myapp

# Watch rollout
kubectl rollout status deployment/myapp-deployment -n myapp

# Check rollout history
kubectl rollout history deployment/myapp-deployment -n myapp

# Rollback if needed
kubectl rollout undo deployment/myapp-deployment -n myapp
```

### Test 2: Scaling

```bash
# Manual scaling
kubectl scale deployment/myapp-deployment --replicas=5 -n myapp

# Check HPA
kubectl get hpa -n myapp

# Generate load to test autoscaling
kubectl run -it --rm load-generator --image=busybox -n myapp -- /bin/sh
# Inside the pod:
while true; do wget -q -O- http://myapp-service; done
```

### Test 3: ConfigMap and Secret Updates

```bash
# Update ConfigMap
kubectl edit configmap myapp-config -n myapp

# Restart deployment to pick up changes
kubectl rollout restart deployment/myapp-deployment -n myapp

# Verify changes
kubectl exec -it <pod-name> -n myapp -- env | grep LOG_LEVEL
```

---

## 📊 Monitoring and Debugging

### View Logs

```bash
# View logs from all pods
kubectl logs -l app=myapp -n myapp --tail=100 -f

# View logs from specific pod
kubectl logs <pod-name> -n myapp

# View previous container logs
kubectl logs <pod-name> -n myapp --previous
```

### Debug Pods

```bash
# Describe pod
kubectl describe pod <pod-name> -n myapp

# Execute commands in pod
kubectl exec -it <pod-name> -n myapp -- /bin/sh

# Check events
kubectl get events -n myapp --sort-by='.lastTimestamp'
```

### Resource Usage

```bash
# Check resource usage
kubectl top nodes
kubectl top pods -n myapp

# Check resource quotas
kubectl describe resourcequota -n myapp
```

---

## 🎯 Success Criteria

- [ ] Minikube cluster running
- [ ] All pods in Running state
- [ ] Service accessible via Ingress
- [ ] ConfigMaps and Secrets working
- [ ] Rolling updates successful
- [ ] Autoscaling functional
- [ ] Helm chart deploys successfully
- [ ] Monitoring and logging working

---

## 📚 Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

## 🎓 What You Learned

✅ Kubernetes cluster setup with Minikube  
✅ Creating and managing Kubernetes resources  
✅ ConfigMaps and Secrets management  
✅ Service discovery and networking  
✅ Ingress configuration  
✅ Horizontal Pod Autoscaling  
✅ Helm chart creation and deployment  
✅ Rolling updates and rollbacks  

---

## 🚀 Next Steps

1. **Project 3**: Provision AWS infrastructure with Terraform
2. Add persistent storage with PVCs
3. Implement StatefulSets for databases
4. Set up service mesh (Istio/Linkerd)
5. Implement GitOps with ArgoCD

---

**Project Status:** ✅ Complete  
**Last Updated:** 2024-01-16