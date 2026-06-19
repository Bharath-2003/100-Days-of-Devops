# Kubernetes for DevOps

## 🎯 Overview

Kubernetes (K8s) is the leading container orchestration platform for automating deployment, scaling, and management of containerized applications. For DevOps engineers, mastering Kubernetes is essential for building cloud-native, scalable infrastructure.

**Duration:** 3 weeks (Weeks 13-15)
**Time Investment:** 4-5 hours/day
**Difficulty:** Intermediate to Advanced
**Prerequisites:** Docker, Linux, Networking basics

---

## 📚 What You'll Learn

### Week 1: Kubernetes Fundamentals
- Kubernetes architecture
- Pods, ReplicaSets, Deployments
- Services and networking
- ConfigMaps and Secrets
- Namespaces and labels

### Week 2: Advanced Kubernetes
- StatefulSets and DaemonSets
- Persistent Volumes
- Ingress controllers
- RBAC and security
- Resource management

### Week 3: Production Kubernetes
- Helm package manager
- Monitoring and logging
- CI/CD with Kubernetes
- Service mesh (Istio)
- Cluster management

---

## 🎓 Important Concepts

### 1. Kubernetes Architecture

```
Kubernetes Cluster Architecture
├─ Control Plane (Master)
│  ├─ API Server (kube-apiserver)
│  │  └─ REST API for all operations
│  │
│  ├─ etcd
│  │  └─ Distributed key-value store
│  │
│  ├─ Scheduler (kube-scheduler)
│  │  └─ Assigns pods to nodes
│  │
│  ├─ Controller Manager (kube-controller-manager)
│  │  ├─ Node Controller
│  │  ├─ Replication Controller
│  │  ├─ Endpoints Controller
│  │  └─ Service Account Controller
│  │
│  └─ Cloud Controller Manager
│     └─ Cloud-specific control logic
│
└─ Worker Nodes
   ├─ kubelet
   │  └─ Node agent
   │
   ├─ kube-proxy
   │  └─ Network proxy
   │
   ├─ Container Runtime
   │  ├─ containerd
   │  ├─ CRI-O
   │  └─ Docker (deprecated)
   │
   └─ Pods
      └─ Containers
```

**Key Components:**
- **API Server:** Central management point
- **etcd:** Cluster state storage
- **Scheduler:** Pod placement decisions
- **Controller Manager:** Maintains desired state
- **kubelet:** Runs containers on nodes
- **kube-proxy:** Network routing

---

### 2. Core Kubernetes Objects

```
Kubernetes Objects Hierarchy
├─ Workloads
│  ├─ Pod (smallest deployable unit)
│  ├─ ReplicaSet (maintains pod replicas)
│  ├─ Deployment (declarative updates)
│  ├─ StatefulSet (stateful applications)
│  ├─ DaemonSet (one pod per node)
│  ├─ Job (run-to-completion)
│  └─ CronJob (scheduled jobs)
│
├─ Services & Networking
│  ├─ Service (stable endpoint)
│  │  ├─ ClusterIP (internal)
│  │  ├─ NodePort (external via node)
│  │  ├─ LoadBalancer (cloud LB)
│  │  └─ ExternalName (DNS)
│  ├─ Ingress (HTTP routing)
│  └─ NetworkPolicy (traffic rules)
│
├─ Configuration
│  ├─ ConfigMap (configuration data)
│  ├─ Secret (sensitive data)
│  └─ PersistentVolumeClaim (storage)
│
└─ Cluster
   ├─ Namespace (virtual cluster)
   ├─ Node (worker machine)
   ├─ PersistentVolume (storage)
   └─ StorageClass (storage types)
```

---

### 3. Pod Lifecycle

```
Pod Lifecycle States
├─ Pending
│  └─ Waiting for scheduling
│
├─ Running
│  └─ At least one container running
│
├─ Succeeded
│  └─ All containers terminated successfully
│
├─ Failed
│  └─ At least one container failed
│
└─ Unknown
   └─ Cannot determine state

Container States
├─ Waiting (not yet running)
├─ Running (executing)
└─ Terminated (finished/failed)

Pod Conditions
├─ PodScheduled
├─ Initialized
├─ ContainersReady
└─ Ready
```

**Pod Spec Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
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
      initialDelaySeconds: 5
      periodSeconds: 5
```

---

### 4. Deployments and ReplicaSets

```
Deployment Strategy
├─ RollingUpdate (default)
│  ├─ maxSurge: 25%
│  └─ maxUnavailable: 25%
│
├─ Recreate
│  └─ Terminate all, then create new
│
└─ Blue-Green (via labels)
   └─ Switch traffic between versions

Deployment → ReplicaSet → Pods
     ↓
  Manages versions
     ↓
  Rollout/Rollback
```

**Deployment Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: ENVIRONMENT
          value: "production"
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: config
        configMap:
          name: nginx-config
```

---

### 5. Services and Networking

```
Service Types
├─ ClusterIP (default)
│  └─ Internal cluster IP
│     └─ 10.96.0.1 (example)
│
├─ NodePort
│  └─ Exposes on each node's IP
│     └─ <NodeIP>:<NodePort>
│
├─ LoadBalancer
│  └─ Cloud provider load balancer
│     └─ External IP assigned
│
└─ ExternalName
   └─ Maps to external DNS
      └─ CNAME record

Service Discovery
├─ DNS (CoreDNS)
│  └─ service-name.namespace.svc.cluster.local
│
└─ Environment Variables
   └─ SERVICE_NAME_SERVICE_HOST
```

**Service Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
```

**Ingress Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

---

### 6. ConfigMaps and Secrets

```
Configuration Management
├─ ConfigMap (non-sensitive)
│  ├─ Environment variables
│  ├─ Command-line arguments
│  ├─ Configuration files
│  └─ Volume mounts
│
└─ Secret (sensitive)
   ├─ Opaque (generic)
   ├─ kubernetes.io/service-account-token
   ├─ kubernetes.io/dockerconfigjson
   └─ kubernetes.io/tls
```

**ConfigMap Example:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Property-like keys
  database_url: "postgres://db:5432/myapp"
  log_level: "info"
  
  # File-like keys
  app.properties: |
    server.port=8080
    server.host=0.0.0.0
    cache.enabled=true
  
  nginx.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://backend:8080;
      }
    }
```

**Secret Example:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  database-password: "mypassword"
  api-key: "sk-1234567890"
data:
  # Base64 encoded
  token: bXktdG9rZW4=
```

**Using ConfigMap and Secret:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    # From ConfigMap
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    # From Secret
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: database-password
    # Mount ConfigMap as volume
    volumeMounts:
    - name: config
      mountPath: /etc/config
    # Mount Secret as volume
    - name: secrets
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: config
    configMap:
      name: app-config
  - name: secrets
    secret:
      secretName: app-secrets
```

---

### 7. Persistent Storage

```
Storage Architecture
├─ PersistentVolume (PV)
│  └─ Cluster-level storage resource
│
├─ PersistentVolumeClaim (PVC)
│  └─ Request for storage
│
├─ StorageClass
│  └─ Dynamic provisioning
│
└─ Volume Types
   ├─ hostPath (node local)
   ├─ emptyDir (pod lifetime)
   ├─ nfs (network file system)
   ├─ cephfs (Ceph)
   ├─ awsElasticBlockStore (EBS)
   ├─ azureDisk (Azure Disk)
   └─ gcePersistentDisk (GCE PD)

Access Modes
├─ ReadWriteOnce (RWO) - single node
├─ ReadOnlyMany (ROX) - multiple nodes read
└─ ReadWriteMany (RWX) - multiple nodes read/write
```

**StorageClass Example:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

**PersistentVolumeClaim Example:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 10Gi
```

**StatefulSet with PVC:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 10Gi
```

---

## 🔨 Hands-On Exercises

### Exercise 1: Set Up Kubernetes Cluster (60 minutes)

**Objective:** Create a local Kubernetes cluster

```bash
# Option 1: Using Minikube
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start --cpus=4 --memory=8192 --driver=docker

# Enable addons
minikube addons enable ingress
minikube addons enable metrics-server
minikube addons enable dashboard

# Access dashboard
minikube dashboard

# Option 2: Using kind (Kubernetes in Docker)
# Install kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create cluster
cat > kind-config.yaml << 'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
EOF

kind create cluster --config kind-config.yaml --name dev-cluster

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client
kubectl cluster-info
kubectl get nodes

# Set up kubectl autocompletion
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc

# Install k9s (terminal UI)
curl -sS https://webinstall.dev/k9s | bash
```

**Challenge:**
Set up production-grade cluster:
1. Multi-master setup
2. External etcd
3. Network policies
4. Pod security policies

---

### Exercise 2: Deploy First Application (45 minutes)

**Objective:** Deploy a complete application stack

```bash
# 1. Create namespace
kubectl create namespace demo

# 2. Create deployment
cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: demo
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
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
EOF

kubectl apply -f deployment.yaml

# 3. Create service
cat > service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: demo
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
EOF

kubectl apply -f service.yaml

# 4. Check deployment
kubectl get deployments -n demo
kubectl get pods -n demo
kubectl get services -n demo

# 5. Scale deployment
kubectl scale deployment nginx-deployment --replicas=5 -n demo

# 6. Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.22 -n demo

# 7. Check rollout status
kubectl rollout status deployment/nginx-deployment -n demo

# 8. View rollout history
kubectl rollout history deployment/nginx-deployment -n demo

# 9. Rollback if needed
kubectl rollout undo deployment/nginx-deployment -n demo

# 10. Port forward to test
kubectl port-forward -n demo svc/nginx-service 8080:80

# Test in another terminal
curl http://localhost:8080

# 11. View logs
kubectl logs -n demo -l app=nginx --tail=50

# 12. Execute command in pod
kubectl exec -n demo -it $(kubectl get pod -n demo -l app=nginx -o jsonpath='{.items[0].metadata.name}') -- /bin/bash

# 13. Describe resources
kubectl describe deployment nginx-deployment -n demo
kubectl describe pod -n demo -l app=nginx

# 14. Clean up
kubectl delete namespace demo
```

**Challenge:**
Deploy multi-tier application:
1. Frontend (React)
2. Backend (Node.js API)
3. Database (PostgreSQL)
4. Redis cache

---

### Exercise 3: ConfigMaps and Secrets (45 minutes)

**Objective:** Manage application configuration

```bash
# 1. Create ConfigMap from literal
kubectl create configmap app-config \
  --from-literal=database_url=postgres://db:5432/myapp \
  --from-literal=log_level=info \
  -n demo

# 2. Create ConfigMap from file
cat > app.properties << 'EOF'
server.port=8080
server.host=0.0.0.0
cache.enabled=true
cache.ttl=3600
EOF

kubectl create configmap app-properties \
  --from-file=app.properties \
  -n demo

# 3. Create ConfigMap from YAML
cat > configmap.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: demo
data:
  nginx.conf: |
    server {
      listen 80;
      server_name _;
      
      location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
      }
      
      location /health {
        return 200 "healthy\n";
      }
    }
EOF

kubectl apply -f configmap.yaml

# 4. Create Secret
kubectl create secret generic app-secrets \
  --from-literal=database-password=mypassword \
  --from-literal=api-key=sk-1234567890 \
  -n demo

# 5. Create TLS Secret
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=app.example.com/O=MyOrg"

kubectl create secret tls app-tls \
  --cert=tls.crt \
  --key=tls.key \
  -n demo

# 6. Create Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=user@example.com \
  -n demo

# 7. Use ConfigMap and Secret in Pod
cat > pod-with-config.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  namespace: demo
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log_level
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: database-password
    volumeMounts:
    - name: config
      mountPath: /etc/nginx
    - name: secrets
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: config
    configMap:
      name: nginx-config
  - name: secrets
    secret:
      secretName: app-secrets
  imagePullSecrets:
  - name: regcred
EOF

kubectl apply -f pod-with-config.yaml

# 8. View ConfigMap
kubectl get configmap app-config -n demo -o yaml

# 9. Edit ConfigMap
kubectl edit configmap app-config -n demo

# 10. Delete resources
kubectl delete configmap --all -n demo
kubectl delete secret --all -n demo
```

**Challenge:**
Implement configuration management:
1. Environment-specific configs
2. Secret rotation
3. External secrets (Vault)
4. Config validation

---

### Exercise 4: Persistent Storage (60 minutes)

**Objective:** Deploy stateful application with persistent storage

```bash
# 1. Create StorageClass
cat > storageclass.yaml << 'EOF'
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF

kubectl apply -f storageclass.yaml

# 2. Create PersistentVolume (for local testing)
cat > pv.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-storage
  hostPath:
    path: /data/mysql
EOF

kubectl apply -f pv.yaml

# 3. Create PersistentVolumeClaim
cat > pvc.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: demo
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-storage
  resources:
    requests:
      storage: 10Gi
EOF

kubectl apply -f pvc.yaml

# 4. Deploy MySQL with PVC
cat > mysql-deployment.yaml << 'EOF'
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: demo
type: Opaque
stringData:
  password: "rootpassword"
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: demo
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: demo
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
          name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        livenessProbe:
          exec:
            command:
            - mysqladmin
            - ping
            - -h
            - localhost
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - mysql
            - -h
            - localhost
            - -e
            - SELECT 1
          initialDelaySeconds: 5
          periodSeconds: 2
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-storage
      resources:
        requests:
          storage: 10Gi
EOF

kubectl apply -f mysql-deployment.yaml

# 5. Verify StatefulSet
kubectl get statefulset -n demo
kubectl get pvc -n demo
kubectl get pv

# 6. Test MySQL connection
kubectl run -it --rm mysql-client --image=mysql:8.0 --restart=Never -n demo -- \
  mysql -h mysql-0.mysql -p

# 7. Create database and table
# In MySQL prompt:
# CREATE DATABASE testdb;
# USE testdb;
# CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50));
# INSERT INTO users VALUES (1, 'John Doe');
# SELECT * FROM users;

# 8. Delete pod and verify data persistence
kubectl delete pod mysql-0 -n demo
kubectl wait --for=condition=ready pod/mysql-0 -n demo --timeout=60s

# 9. Verify data still exists
kubectl run -it --rm mysql-client --image=mysql:8.0 --restart=Never -n demo -- \
  mysql -h mysql-0.mysql -p -e "SELECT * FROM testdb.users;"

# 10. Scale StatefulSet
kubectl scale statefulset mysql --replicas=3 -n demo

# 11. View PVCs created
kubectl get pvc -n demo
```

**Challenge:**
Implement production storage:
1. Database replication
2. Backup and restore
3. Volume snapshots
4. Storage monitoring

---

### Exercise 5: Ingress and Load Balancing (60 minutes)

**Objective:** Set up ingress controller and routing

```bash
# 1. Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Wait for ingress controller
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s

# 2. Deploy sample applications
cat > apps.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
  namespace: demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: app
        image: hashicorp/http-echo
        args:
        - "-text=App 1"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: app1-service
  namespace: demo
spec:
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 5678
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
  namespace: demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: app
        image: hashicorp/http-echo
        args:
        - "-text=App 2"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: app2-service
  namespace: demo
spec:
  selector:
    app: app2
  ports:
  - port: 80
    targetPort: 5678
EOF

kubectl apply -f apps.yaml

# 3. Create Ingress resource
cat > ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  namespace: demo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - host: app.local
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
EOF

kubectl apply -f ingress.yaml

# 4. Test ingress
# Add to /etc/hosts: 127.0.0.1 app.local
curl http://app.local/app1
curl http://app.local/app2

# 5. Create TLS Ingress
# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Create ClusterIssuer
cat > cluster-issuer.yaml << 'EOF'
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: nginx
EOF

kubectl apply -f cluster-issuer.yaml

# 6. Create TLS Ingress
cat > tls-ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  namespace: demo
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-staging"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
EOF

kubectl apply -f tls-ingress.yaml

# 7. View ingress
kubectl get ingress -n demo
kubectl describe ingress demo-ingress -n demo
```

**Challenge:**
Advanced ingress setup:
1. Rate limiting
2. Authentication
3. Canary deployments
4. A/B testing

---

## 🎯 Practice Projects

### Project 1: Microservices Platform
Deploy complete microservices:
- API Gateway
- Multiple services
- Service mesh (Istio)
- Distributed tracing
- Centralized logging
- Monitoring with Prometheus
- CI/CD integration

### Project 2: Stateful Application
Deploy production database:
- MySQL/PostgreSQL cluster
- Replication setup
- Automated backups
- Point-in-time recovery
- Monitoring and alerting
- Performance tuning

### Project 3: Multi-Tenant Platform
Build multi-tenant system:
- Namespace isolation
- Resource quotas
- Network policies
- RBAC configuration
- Tenant-specific ingress
- Cost allocation

### Project 4: GitOps Workflow
Implement GitOps:
- ArgoCD/Flux setup
- Git as source of truth
- Automated deployments
- Rollback capability
- Multi-environment
- Secret management

---

## 📝 Best Practices

### 1. Resource Management
- Set resource requests and limits
- Use HPA for auto-scaling
- Implement pod disruption budgets
- Use node affinity/anti-affinity
- Monitor resource usage
- Right-size workloads

### 2. Security
- Use RBAC
- Enable pod security policies
- Scan images for vulnerabilities
- Use network policies
- Encrypt secrets
- Regular security audits
- Least privilege principle

### 3. High Availability
- Deploy across multiple zones
- Use pod anti-affinity
- Implement health checks
- Use StatefulSets for stateful apps
- Configure pod disruption budgets
- Regular disaster recovery testing

### 4. Monitoring
- Use Prometheus for metrics
- Implement logging (ELK/Loki)
- Set up alerting
- Monitor cluster health
- Track application performance
- Use distributed tracing

---

## ✅ Completion Checklist

### Week 1
- [ ] Understand K8s architecture
- [ ] Deploy pods and deployments
- [ ] Create services
- [ ] Use ConfigMaps and Secrets
- [ ] Work with namespaces

### Week 2
- [ ] Deploy StatefulSets
- [ ] Configure persistent storage
- [ ] Set up ingress
- [ ] Implement RBAC
- [ ] Manage resources

### Week 3
- [ ] Use Helm charts
- [ ] Set up monitoring
- [ ] Implement CI/CD
- [ ] Deploy service mesh
- [ ] Manage production cluster

---

**Next:** [Terraform →](../terraform/README.md)