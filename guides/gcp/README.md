# Google Cloud Platform (GCP) for DevOps

## 🎯 Overview

Google Cloud Platform is a comprehensive cloud computing platform with 100+ services. For DevOps engineers, mastering GCP is essential for building scalable, data-driven infrastructure with cutting-edge technologies.

**Duration:** 6 weeks (Weeks 41-46)
**Time Investment:** 4-5 hours/day
**Difficulty:** Intermediate to Advanced
**Prerequisites:** Linux, Networking, Basic cloud concepts

---

## 📚 What You'll Learn

### Week 1: GCP Fundamentals
- GCP account setup and projects
- gcloud CLI and Cloud Shell
- IAM and service accounts
- Compute Engine
- Cloud Storage basics

### Week 2: Networking
- Virtual Private Cloud (VPC)
- Subnets and firewall rules
- Cloud Load Balancing
- Cloud CDN
- Cloud Interconnect

### Week 3: Storage and Databases
- Cloud Storage (buckets and objects)
- Cloud SQL
- Cloud Spanner
- Firestore
- BigQuery basics

### Week 4: Containers and Kubernetes
- Container Registry
- Cloud Run
- Google Kubernetes Engine (GKE)
- Anthos
- Service Mesh

### Week 5: Serverless and Data
- Cloud Functions
- Cloud Pub/Sub
- Cloud Dataflow
- BigQuery advanced
- Cloud Composer

### Week 6: DevOps and Monitoring
- Cloud Build
- Cloud Deploy
- Artifact Registry
- Cloud Monitoring
- Cloud Logging

---

## 🎓 Important Concepts

### 1. GCP Global Infrastructure

```
GCP Global Infrastructure
├─ Regions (35+)
│  ├─ us-central1 (Iowa)
│  ├─ us-east1 (South Carolina)
│  ├─ europe-west1 (Belgium)
│  ├─ asia-south1 (Mumbai)
│  └─ ...
│
├─ Zones (106+)
│  ├─ us-central1-a
│  ├─ us-central1-b
│  ├─ us-central1-c
│  └─ ...
│
└─ Edge Locations (140+)
   └─ Cloud CDN
```

**Key Concepts:**
- **Region:** Independent geographic area with multiple zones
- **Zone:** Isolated location within a region
- **Multi-region:** Large geographic area (US, EU, Asia)
- **Edge Location:** CDN endpoint for Cloud CDN

**Best Practices:**
- Deploy across multiple zones for high availability
- Use multi-region for global applications
- Choose region based on latency and data residency
- Use Cloud CDN for content delivery

---

### 2. IAM and Resource Hierarchy

```
GCP Resource Hierarchy
├─ Organization
│  └─ Folders
│     └─ Projects
│        └─ Resources
│
IAM Model
├─ Who (Identity)
│  ├─ Google Account
│  ├─ Service Account
│  ├─ Google Group
│  └─ Cloud Identity Domain
│
├─ Can do what (Role)
│  ├─ Primitive Roles (Owner, Editor, Viewer)
│  ├─ Predefined Roles
│  └─ Custom Roles
│
└─ On which resource (Scope)
   ├─ Organization
   ├─ Folder
   ├─ Project
   └─ Resource
```

**Service Accounts:**
```bash
# Create service account
gcloud iam service-accounts create my-sa \
  --display-name "My Service Account"

# Grant role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:my-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Create key
gcloud iam service-accounts keys create key.json \
  --iam-account=my-sa@PROJECT_ID.iam.gserviceaccount.com
```

**Common Roles:**
- **roles/owner:** Full access to all resources
- **roles/editor:** Modify resources
- **roles/viewer:** Read-only access
- **roles/compute.admin:** Manage Compute Engine
- **roles/container.admin:** Manage GKE clusters

---

### 3. Compute Engine

```
Compute Engine Components
├─ VM Instances
│  ├─ Machine Types
│  │  ├─ General Purpose (E2, N1, N2, N2D)
│  │  ├─ Compute Optimized (C2, C2D)
│  │  ├─ Memory Optimized (M1, M2)
│  │  └─ Accelerator Optimized (A2)
│  │
│  ├─ Images
│  │  ├─ Public Images (Debian, Ubuntu, CentOS)
│  │  ├─ Custom Images
│  │  └─ Container-Optimized OS
│  │
│  └─ Disks
│     ├─ Persistent Disks (Standard, SSD, Balanced)
│     ├─ Local SSDs
│     └─ Snapshots
│
├─ Instance Groups
│  ├─ Managed Instance Groups (MIG)
│  └─ Unmanaged Instance Groups
│
└─ Instance Templates
```

**Machine Type Families:**
- **E2:** Cost-optimized, burstable
- **N2/N2D:** Balanced price/performance
- **C2/C2D:** Compute-intensive workloads
- **M1/M2:** Memory-intensive workloads
- **A2:** GPU workloads (ML/AI)

**High Availability:**
```
Regional MIG
├─ Zone A (33% instances)
├─ Zone B (33% instances)
└─ Zone C (34% instances)

Auto-scaling
├─ CPU utilization
├─ HTTP load balancing
├─ Cloud Monitoring metrics
└─ Schedule-based
```

---

### 4. Virtual Private Cloud (VPC)

```
VPC Architecture
├─ VPC Network (Global)
│  ├─ Subnets (Regional)
│  │  ├─ us-central1: 10.128.0.0/20
│  │  ├─ europe-west1: 10.132.0.0/20
│  │  └─ asia-east1: 10.140.0.0/20
│  │
│  ├─ Firewall Rules
│  │  ├─ Ingress Rules
│  │  └─ Egress Rules
│  │
│  ├─ Routes
│  │  ├─ System Routes
│  │  └─ Custom Routes
│  │
│  └─ VPC Peering
│
└─ Shared VPC
   └─ Host Project → Service Projects
```

**Firewall Rules:**
```
Priority | Direction | Action | Source      | Target | Ports
---------|-----------|--------|-------------|--------|-------
1000     | Ingress   | Allow  | 0.0.0.0/0   | web    | 80,443
1000     | Ingress   | Allow  | 10.0.0.0/8  | all    | all
2000     | Ingress   | Allow  | 0.0.0.0/0   | ssh    | 22
65535    | Ingress   | Deny   | 0.0.0.0/0   | all    | all
```

**Load Balancing Options:**
- **Global HTTP(S):** Layer 7, global
- **Global SSL Proxy:** Layer 4, SSL/TLS
- **Global TCP Proxy:** Layer 4, TCP
- **Regional Network:** Layer 4, regional
- **Regional Internal:** Layer 4, internal

---

### 5. Cloud Storage

```
Cloud Storage Classes
├─ Standard
│  └─ Frequently accessed data
│
├─ Nearline
│  └─ Accessed < once/month
│
├─ Coldline
│  └─ Accessed < once/quarter
│
└─ Archive
   └─ Accessed < once/year

Storage Locations
├─ Multi-region (US, EU, ASIA)
├─ Dual-region (specific pairs)
└─ Region (single region)
```

**Bucket Operations:**
```bash
# Create bucket
gsutil mb -c STANDARD -l us-central1 gs://my-bucket

# Upload file
gsutil cp file.txt gs://my-bucket/

# Download file
gsutil cp gs://my-bucket/file.txt .

# Sync directory
gsutil rsync -r ./local-dir gs://my-bucket/remote-dir

# Set lifecycle policy
gsutil lifecycle set lifecycle.json gs://my-bucket
```

**Lifecycle Policy Example:**
```json
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
        "condition": {"age": 30}
      },
      {
        "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
        "condition": {"age": 90}
      },
      {
        "action": {"type": "Delete"},
        "condition": {"age": 365}
      }
    ]
  }
}
```

---

### 6. Google Kubernetes Engine (GKE)

```
GKE Architecture
├─ Control Plane (Google-managed)
│  ├─ API Server
│  ├─ Scheduler
│  └─ Controller Manager
│
└─ Node Pools (Customer-managed)
   ├─ Default Node Pool
   └─ Custom Node Pools
      ├─ Standard Nodes
      ├─ Preemptible Nodes
      └─ Spot Nodes
```

**Cluster Types:**
- **Zonal:** Single zone, lower cost
- **Regional:** Multi-zone, high availability
- **Autopilot:** Fully managed, hands-off
- **Standard:** More control, manual management

**GKE Features:**
```
Autopilot Mode
├─ Automatic node provisioning
├─ Automatic scaling
├─ Automatic security updates
└─ Pay-per-pod pricing

Standard Mode
├─ Node pool management
├─ Custom machine types
├─ GPU/TPU support
└─ More configuration options
```

**Workload Identity:**
```bash
# Enable Workload Identity
gcloud container clusters update CLUSTER_NAME \
  --workload-pool=PROJECT_ID.svc.id.goog

# Create Kubernetes service account
kubectl create serviceaccount KSA_NAME

# Bind to GCP service account
gcloud iam service-accounts add-iam-policy-binding GSA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]"
```

---

### 7. Cloud Build and CI/CD

```
Cloud Build Pipeline
├─ Source (Cloud Source Repositories, GitHub, Bitbucket)
├─ Build Steps
│  ├─ Build container image
│  ├─ Run tests
│  ├─ Security scanning
│  └─ Push to Artifact Registry
│
└─ Deploy
   ├─ Cloud Run
   ├─ GKE
   ├─ Compute Engine
   └─ App Engine
```

**cloudbuild.yaml Example:**
```yaml
steps:
  # Build Docker image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA', '.']
  
  # Run tests
  - name: 'gcr.io/cloud-builders/docker'
    args: ['run', 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA', 'npm', 'test']
  
  # Push to Container Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA']
  
  # Deploy to GKE
  - name: 'gcr.io/cloud-builders/kubectl'
    args:
      - 'set'
      - 'image'
      - 'deployment/myapp'
      - 'myapp=gcr.io/$PROJECT_ID/myapp:$SHORT_SHA'
    env:
      - 'CLOUDSDK_COMPUTE_REGION=us-central1'
      - 'CLOUDSDK_CONTAINER_CLUSTER=my-cluster'

images:
  - 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA'
```

---

## 🔨 Hands-On Exercises

### Exercise 1: gcloud CLI Setup (30 minutes)

**Objective:** Install and configure gcloud CLI

```bash
# 1. Install gcloud CLI (Linux)
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# 2. Initialize gcloud
gcloud init

# 3. Login
gcloud auth login

# 4. List projects
gcloud projects list

# 5. Set active project
gcloud config set project PROJECT_ID

# 6. Set default region and zone
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# 7. View configuration
gcloud config list

# 8. Create new configuration
gcloud config configurations create dev
gcloud config configurations activate dev

# 9. Enable APIs
gcloud services enable compute.googleapis.com
gcloud services enable container.googleapis.com
gcloud services enable cloudbuild.googleapis.com

# 10. List enabled services
gcloud services list --enabled
```

**Challenge:**
Create a script that:
1. Sets up multiple configurations (dev, staging, prod)
2. Enables required APIs
3. Creates service accounts
4. Generates and stores keys securely

---

### Exercise 2: Deploy Compute Engine VM (60 minutes)

**Objective:** Create and manage Compute Engine instances

```bash
# 1. Create VPC network
gcloud compute networks create vpc-devops \
  --subnet-mode=custom

# 2. Create subnet
gcloud compute networks subnets create subnet-web \
  --network=vpc-devops \
  --region=us-central1 \
  --range=10.0.1.0/24

# 3. Create firewall rules
gcloud compute firewall-rules create allow-ssh \
  --network=vpc-devops \
  --allow=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=ssh

gcloud compute firewall-rules create allow-http \
  --network=vpc-devops \
  --allow=tcp:80,tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=web

# 4. Create startup script
cat > startup.sh << 'EOF'
#!/bin/bash
apt-get update
apt-get install -y nginx
systemctl start nginx
systemctl enable nginx
echo "<h1>Hello from GCP!</h1>" > /var/www/html/index.html
EOF

# 5. Create VM instance
gcloud compute instances create vm-web-01 \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --network=vpc-devops \
  --subnet=subnet-web \
  --tags=ssh,web \
  --metadata-from-file=startup-script=startup.sh \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=20GB \
  --boot-disk-type=pd-standard

# 6. Get instance details
gcloud compute instances describe vm-web-01 \
  --zone=us-central1-a

# 7. Get external IP
EXTERNAL_IP=$(gcloud compute instances describe vm-web-01 \
  --zone=us-central1-a \
  --format='get(networkInterfaces[0].accessConfigs[0].natIP)')

echo "VM External IP: $EXTERNAL_IP"

# 8. SSH to instance
gcloud compute ssh vm-web-01 --zone=us-central1-a

# 9. Create custom image
gcloud compute images create custom-web-image \
  --source-disk=vm-web-01 \
  --source-disk-zone=us-central1-a \
  --family=custom-web

# 10. Stop instance
gcloud compute instances stop vm-web-01 --zone=us-central1-a

# 11. Start instance
gcloud compute instances start vm-web-01 --zone=us-central1-a

# 12. Delete instance
gcloud compute instances delete vm-web-01 \
  --zone=us-central1-a \
  --quiet
```

**Challenge:**
Create a managed instance group with:
1. Instance template
2. Auto-scaling (2-10 instances)
3. Health checks
4. Load balancer

---

### Exercise 3: Cloud Storage Operations (45 minutes)

**Objective:** Work with Cloud Storage buckets

```bash
# 1. Create bucket
gsutil mb -c STANDARD -l us-central1 gs://devops-bucket-$RANDOM

BUCKET_NAME=$(gsutil ls | grep devops-bucket | sed 's/gs:\/\///' | sed 's/\///')

# 2. Upload files
echo "Log entry 1" > app.log
gsutil cp app.log gs://$BUCKET_NAME/logs/

# 3. Upload directory
mkdir -p data
echo "Data 1" > data/file1.txt
echo "Data 2" > data/file2.txt
gsutil cp -r data gs://$BUCKET_NAME/

# 4. List objects
gsutil ls gs://$BUCKET_NAME/
gsutil ls -r gs://$BUCKET_NAME/**

# 5. Download file
gsutil cp gs://$BUCKET_NAME/logs/app.log downloaded.log

# 6. Set object metadata
gsutil setmeta -h "Content-Type:text/plain" \
  -h "Cache-Control:public, max-age=3600" \
  gs://$BUCKET_NAME/logs/app.log

# 7. Make object public
gsutil acl ch -u AllUsers:R gs://$BUCKET_NAME/logs/app.log

# 8. Create lifecycle policy
cat > lifecycle.json << 'EOF'
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
        "condition": {"age": 30, "matchesPrefix": ["logs/"]}
      },
      {
        "action": {"type": "Delete"},
        "condition": {"age": 90, "matchesPrefix": ["logs/"]}
      }
    ]
  }
}
EOF

gsutil lifecycle set lifecycle.json gs://$BUCKET_NAME

# 9. Enable versioning
gsutil versioning set on gs://$BUCKET_NAME

# 10. Sync directories
gsutil rsync -r ./local-dir gs://$BUCKET_NAME/backup/

# 11. Sign URL (1 hour expiry)
gsutil signurl -d 1h key.json gs://$BUCKET_NAME/logs/app.log

# 12. Delete objects
gsutil rm gs://$BUCKET_NAME/logs/app.log

# 13. Delete bucket
gsutil rm -r gs://$BUCKET_NAME
```

**Challenge:**
Create a backup solution that:
1. Backs up to Cloud Storage daily
2. Uses lifecycle policies
3. Replicates to different region
4. Sends notifications via Pub/Sub

---

### Exercise 4: Deploy GKE Cluster (90 minutes)

**Objective:** Create and manage GKE cluster

```bash
# 1. Create GKE cluster
gcloud container clusters create gke-devops-cluster \
  --zone=us-central1-a \
  --num-nodes=2 \
  --machine-type=e2-medium \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=5 \
  --enable-autorepair \
  --enable-autoupgrade \
  --workload-pool=PROJECT_ID.svc.id.goog

# 2. Get credentials
gcloud container clusters get-credentials gke-devops-cluster \
  --zone=us-central1-a

# 3. Verify connection
kubectl get nodes

# 4. Create namespace
kubectl create namespace production

# 5. Deploy application
cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: production
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
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: production
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
EOF

kubectl apply -f deployment.yaml

# 6. Wait for external IP
kubectl get service nginx-service -n production --watch

# 7. Scale deployment
kubectl scale deployment nginx-deployment \
  --replicas=5 \
  -n production

# 8. Create HPA
kubectl autoscale deployment nginx-deployment \
  --cpu-percent=50 \
  --min=3 \
  --max=10 \
  -n production

# 9. View logs
kubectl logs -l app=nginx -n production --tail=50

# 10. Add node pool
gcloud container node-pools create high-mem-pool \
  --cluster=gke-devops-cluster \
  --zone=us-central1-a \
  --machine-type=e2-highmem-2 \
  --num-nodes=1 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=3

# 11. View cluster info
gcloud container clusters describe gke-devops-cluster \
  --zone=us-central1-a
```

**Challenge:**
Deploy a microservices application with:
1. Multiple services
2. Ingress controller
3. TLS certificates
4. Monitoring with Cloud Monitoring

---

### Exercise 5: Cloud Build Pipeline (60 minutes)

**Objective:** Create CI/CD pipeline with Cloud Build

```bash
# 1. Create Cloud Source Repository
gcloud source repos create my-app

# 2. Clone repository
gcloud source repos clone my-app
cd my-app

# 3. Create application
cat > app.py << 'EOF'
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Cloud Build!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
EOF

cat > requirements.txt << 'EOF'
Flask==2.3.0
EOF

cat > Dockerfile << 'EOF'
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
EXPOSE 8080
CMD ["python", "app.py"]
EOF

# 4. Create Cloud Build config
cat > cloudbuild.yaml << 'EOF'
steps:
  # Build the container image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA', '.']
  
  # Push to Container Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA']
  
  # Deploy to Cloud Run
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'myapp'
      - '--image=gcr.io/$PROJECT_ID/myapp:$SHORT_SHA'
      - '--region=us-central1'
      - '--platform=managed'
      - '--allow-unauthenticated'

images:
  - 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA'
EOF

# 5. Commit and push
git add .
git commit -m "Initial commit"
git push origin master

# 6. Create build trigger
gcloud builds triggers create cloud-source-repositories \
  --repo=my-app \
  --branch-pattern="^master$" \
  --build-config=cloudbuild.yaml

# 7. Manual build
gcloud builds submit --config cloudbuild.yaml .

# 8. List builds
gcloud builds list --limit=5

# 9. View build logs
BUILD_ID=$(gcloud builds list --limit=1 --format='value(id)')
gcloud builds log $BUILD_ID

# 10. Get Cloud Run URL
gcloud run services describe myapp \
  --region=us-central1 \
  --format='value(status.url)'
```

**Challenge:**
Create a complete CI/CD pipeline with:
1. Multiple environments (dev, staging, prod)
2. Automated testing
3. Security scanning
4. Approval gates

---

### Exercise 6: Cloud Monitoring (45 minutes)

**Objective:** Set up monitoring and alerting

```bash
# 1. Create uptime check
gcloud monitoring uptime create web-check \
  --resource-type=uptime-url \
  --host=example.com \
  --path=/ \
  --check-interval=60s

# 2. Create notification channel
gcloud alpha monitoring channels create \
  --display-name="Email Alerts" \
  --type=email \
  --channel-labels=email_address=admin@example.com

# 3. Create alert policy
cat > alert-policy.json << 'EOF'
{
  "displayName": "High CPU Usage",
  "conditions": [{
    "displayName": "CPU usage above 80%",
    "conditionThreshold": {
      "filter": "resource.type=\"gce_instance\" AND metric.type=\"compute.googleapis.com/instance/cpu/utilization\"",
      "comparison": "COMPARISON_GT",
      "thresholdValue": 0.8,
      "duration": "300s"
    }
  }],
  "combiner": "OR",
  "enabled": true
}
EOF

gcloud alpha monitoring policies create --policy-from-file=alert-policy.json

# 4. Create custom metric
gcloud logging metrics create custom_error_count \
  --description="Count of custom errors" \
  --log-filter='severity="ERROR" AND textPayload:"CustomError"'

# 5. Create dashboard
cat > dashboard.json << 'EOF'
{
  "displayName": "My Dashboard",
  "mosaicLayout": {
    "columns": 12,
    "tiles": [{
      "width": 6,
      "height": 4,
      "widget": {
        "title": "CPU Utilization",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "resource.type=\"gce_instance\" AND metric.type=\"compute.googleapis.com/instance/cpu/utilization\""
              }
            }
          }]
        }
      }
    }]
  }
}
EOF

gcloud monitoring dashboards create --config-from-file=dashboard.json

# 6. View logs
gcloud logging read "resource.type=gce_instance" \
  --limit=10 \
  --format=json

# 7. Create log sink
gcloud logging sinks create my-sink \
  storage.googleapis.com/my-logs-bucket \
  --log-filter='severity>=ERROR'
```

---

## 🎯 Practice Projects

### Project 1: Three-Tier Web Application
Deploy complete web application:
- Cloud Load Balancing
- Managed Instance Group
- Cloud SQL (MySQL/PostgreSQL)
- Cloud Storage for static assets
- Cloud CDN
- Cloud Armor for security
- Cloud Monitoring

### Project 2: Microservices on GKE
Build microservices platform:
- GKE Autopilot cluster
- Multiple microservices
- Istio service mesh
- Cloud Pub/Sub for messaging
- Cloud Firestore for data
- Cloud Build for CI/CD
- Cloud Trace for distributed tracing

### Project 3: Serverless Data Pipeline
Create data processing pipeline:
- Cloud Functions for processing
- Cloud Pub/Sub for events
- Cloud Dataflow for ETL
- BigQuery for analytics
- Cloud Scheduler for automation
- Cloud Monitoring for observability

### Project 4: Multi-Region Deployment
Implement global application:
- Multi-region GKE clusters
- Global Load Balancing
- Cloud Spanner (global database)
- Cloud CDN
- Cloud DNS
- Disaster recovery setup

---

## 📝 Best Practices

### 1. Security
- Use service accounts with minimal permissions
- Enable VPC Service Controls
- Use Secret Manager for secrets
- Enable Binary Authorization
- Use Cloud Armor for DDoS protection
- Regular security scanning
- Enable audit logging

### 2. Cost Optimization
- Use committed use discounts
- Use preemptible/spot instances
- Right-size resources
- Use sustained use discounts
- Delete unused resources
- Use Cloud Billing budgets
- Enable recommendations

### 3. High Availability
- Deploy across multiple zones
- Use regional resources
- Implement health checks
- Use managed instance groups
- Enable auto-scaling
- Regular disaster recovery testing

### 4. Monitoring
- Enable Cloud Monitoring
- Set up alerting policies
- Use Cloud Logging
- Implement distributed tracing
- Create custom dashboards
- Use Cloud Profiler
- Enable Error Reporting

---

## ✅ Completion Checklist

### Week 1
- [ ] Set up GCP account
- [ ] Configure gcloud CLI
- [ ] Create Compute Engine VMs
- [ ] Work with Cloud Storage
- [ ] Understand IAM

### Week 2
- [ ] Create VPC networks
- [ ] Configure firewall rules
- [ ] Deploy load balancers
- [ ] Set up Cloud CDN
- [ ] Understand networking

### Week 3
- [ ] Work with Cloud Storage
- [ ] Create Cloud SQL instances
- [ ] Use Cloud Spanner
- [ ] Query BigQuery
- [ ] Implement backups

### Week 4
- [ ] Deploy GKE cluster
- [ ] Use Container Registry
- [ ] Deploy to Cloud Run
- [ ] Configure Workload Identity
- [ ] Implement service mesh

### Week 5
- [ ] Create Cloud Functions
- [ ] Use Cloud Pub/Sub
- [ ] Build data pipelines
- [ ] Advanced BigQuery
- [ ] Use Cloud Composer

### Week 6
- [ ] Build CI/CD pipelines
- [ ] Use Cloud Build
- [ ] Configure monitoring
- [ ] Set up logging
- [ ] Complete projects

---

**Next:** [Prometheus →](../prometheus/README.md)