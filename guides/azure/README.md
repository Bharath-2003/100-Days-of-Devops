# Microsoft Azure for DevOps

## 🎯 Overview

Microsoft Azure is a comprehensive cloud computing platform with 200+ services. For DevOps engineers, mastering Azure is essential for building enterprise-grade, scalable infrastructure with strong hybrid cloud capabilities.

**Duration:** 6 weeks (Weeks 35-40)
**Time Investment:** 4-5 hours/day
**Difficulty:** Intermediate to Advanced
**Prerequisites:** Linux, Networking, Basic cloud concepts

---

## 📚 What You'll Learn

### Week 1: Azure Fundamentals
- Azure account setup and subscriptions
- Azure CLI and PowerShell
- Resource Groups and ARM
- Azure Virtual Machines
- Azure Storage basics

### Week 2: Networking
- Virtual Networks (VNet)
- Subnets and NSGs
- Load Balancers
- Application Gateway
- VPN and ExpressRoute

### Week 3: Storage and Databases
- Azure Storage (Blob, File, Queue, Table)
- Azure SQL Database
- Cosmos DB
- Azure Cache for Redis
- Backup and Recovery

### Week 4: Identity and Security
- Azure Active Directory
- RBAC and Managed Identities
- Key Vault
- Security Center
- Azure Policy

### Week 5: Containers and Kubernetes
- Azure Container Registry
- Azure Container Instances
- Azure Kubernetes Service (AKS)
- Helm and GitOps
- Service Mesh

### Week 6: DevOps and Automation
- Azure DevOps Services
- Azure Pipelines
- ARM Templates and Bicep
- Terraform on Azure
- Monitoring with Azure Monitor

---

## 🎓 Important Concepts

### 1. Azure Global Infrastructure

```
Azure Global Infrastructure
├─ Regions (60+)
│  ├─ East US - eastus
│  ├─ West Europe - westeurope
│  ├─ Southeast Asia - southeastasia
│  ├─ Central India - centralindia
│  └─ ...
│
├─ Availability Zones (Multiple per region)
│  ├─ Zone 1
│  ├─ Zone 2
│  └─ Zone 3
│
└─ Region Pairs
   └─ Disaster recovery pairing
```

**Key Concepts:**
- **Region:** Geographic area with multiple datacenters
- **Availability Zone:** Isolated datacenter within region
- **Region Pair:** Paired regions for disaster recovery
- **Geography:** Data residency boundary

**Best Practices:**
- Deploy across multiple availability zones
- Use region pairs for disaster recovery
- Choose region based on latency and compliance
- Use Azure Front Door for global distribution

---

### 2. Azure Resource Manager (ARM)

```
ARM Hierarchy
├─ Management Groups
│  └─ Subscriptions
│     └─ Resource Groups
│        └─ Resources
│
ARM Template Structure
├─ $schema
├─ contentVersion
├─ parameters
├─ variables
├─ resources
└─ outputs
```

**Key Concepts:**
- **Resource Group:** Logical container for resources
- **ARM Template:** JSON-based infrastructure as code
- **Deployment:** Process of creating/updating resources
- **Tags:** Metadata for organization and billing

**Example ARM Template:**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "mystorageaccount",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2"
    }
  ]
}
```

---

### 3. Azure Virtual Machines

```
VM Components
├─ VM Size (CPU, RAM, Disk)
├─ Operating System (Windows/Linux)
├─ Managed Disks
│  ├─ OS Disk
│  └─ Data Disks
├─ Network Interface
├─ Public/Private IP
├─ Network Security Group
└─ Availability Set/Zone
```

**VM Size Series:**
- **B-series:** Burstable, cost-effective
- **D-series:** General purpose
- **E-series:** Memory optimized
- **F-series:** Compute optimized
- **N-series:** GPU enabled

**Availability Options:**
```
Availability Sets (99.95% SLA)
├─ Fault Domains (rack isolation)
└─ Update Domains (maintenance isolation)

Availability Zones (99.99% SLA)
├─ Zone 1
├─ Zone 2
└─ Zone 3
```

---

### 4. Azure Virtual Network (VNet)

```
VNet Architecture
├─ Address Space: 10.0.0.0/16
├─ Subnets
│  ├─ Web Subnet: 10.0.1.0/24
│  ├─ App Subnet: 10.0.2.0/24
│  └─ DB Subnet: 10.0.3.0/24
├─ Network Security Groups (NSGs)
├─ Route Tables
└─ Service Endpoints
```

**NSG Rules:**
```
Priority | Name        | Port | Protocol | Source   | Action
---------|-------------|------|----------|----------|-------
100      | Allow-SSH   | 22   | TCP      | MyIP     | Allow
200      | Allow-HTTP  | 80   | TCP      | Internet | Allow
300      | Allow-HTTPS | 443  | TCP      | Internet | Allow
65000    | Deny-All    | *    | *        | *        | Deny
```

**Key Services:**
- **Load Balancer:** Layer 4 load balancing
- **Application Gateway:** Layer 7 with WAF
- **VPN Gateway:** Site-to-site VPN
- **ExpressRoute:** Private connection
- **Azure Firewall:** Managed firewall service

---

### 5. Azure Storage

```
Storage Account Types
├─ Blob Storage (objects)
│  ├─ Block Blobs (files, backups)
│  ├─ Append Blobs (logs)
│  └─ Page Blobs (VHD disks)
├─ File Storage (SMB shares)
├─ Queue Storage (messaging)
├─ Table Storage (NoSQL)
└─ Disk Storage (VM disks)
```

**Storage Tiers:**
- **Hot:** Frequently accessed data
- **Cool:** Infrequently accessed (30+ days)
- **Archive:** Rarely accessed (180+ days)

**Redundancy Options:**
```
Local:
├─ LRS (Locally Redundant) - 3 copies, 1 datacenter
└─ ZRS (Zone Redundant) - 3 copies, 3 zones

Geo:
├─ GRS (Geo Redundant) - LRS + geo replication
├─ GZRS (Geo Zone Redundant) - ZRS + geo
├─ RA-GRS (Read Access GRS)
└─ RA-GZRS (Read Access GZRS)
```

---

### 6. Azure Active Directory (Azure AD)

```
Azure AD Structure
├─ Tenant
│  ├─ Users
│  ├─ Groups
│  ├─ Applications
│  └─ Service Principals
│
RBAC Model
├─ Security Principal (Who)
├─ Role Definition (What)
└─ Scope (Where)
```

**Built-in Roles:**
- **Owner:** Full access including access management
- **Contributor:** Create and manage resources
- **Reader:** View resources only
- **Custom Roles:** Tailored permissions

**Managed Identities:**
```bash
# System-assigned (tied to resource)
az vm identity assign --name myVM --resource-group myRG

# User-assigned (independent)
az identity create --name myIdentity --resource-group myRG
az vm identity assign --name myVM --identities myIdentity
```

---

### 7. Azure Kubernetes Service (AKS)

```
AKS Architecture
├─ Control Plane (Microsoft-managed)
│  ├─ API Server
│  ├─ Scheduler
│  └─ Controller Manager
│
└─ Node Pools (Customer-managed)
   ├─ System Node Pool
   └─ User Node Pools
```

**Key Features:**
- Managed Kubernetes control plane
- Integrated with Azure AD
- Azure Monitor for containers
- Virtual nodes (serverless)
- Azure Policy for Kubernetes
- GitOps with Flux

**Networking Options:**
- **kubenet:** Basic networking
- **Azure CNI:** Advanced networking with VNet integration

---

## 🔨 Hands-On Exercises

### Exercise 1: Azure CLI Setup (30 minutes)

**Objective:** Install and configure Azure CLI

```bash
# 1. Install Azure CLI (Linux)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# 2. Login to Azure
az login

# 3. List subscriptions
az account list --output table

# 4. Set active subscription
az account set --subscription "Your-Subscription-Name"

# 5. Show current subscription
az account show --output table

# 6. List all regions
az account list-locations --output table

# 7. Create resource group
az group create \
  --name rg-devops-demo \
  --location eastus \
  --tags Environment=Dev Project=DevOps

# 8. List resource groups
az group list --output table

# 9. Show resource group details
az group show --name rg-devops-demo

# 10. Export resource group as template
az group export \
  --name rg-devops-demo \
  --output json > rg-template.json
```

**Challenge:**
Create a script that:
1. Lists all resource groups
2. Shows resource count per group
3. Calculates total cost per group
4. Exports to CSV

---

### Exercise 2: Deploy Virtual Machine (60 minutes)

**Objective:** Create and manage Azure VMs

```bash
# 1. Create VNet
az network vnet create \
  --resource-group rg-devops-demo \
  --name vnet-devops \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-web \
  --subnet-prefix 10.0.1.0/24

# 2. Create NSG
az network nsg create \
  --resource-group rg-devops-demo \
  --name nsg-web

# 3. Add NSG rules
az network nsg rule create \
  --resource-group rg-devops-demo \
  --nsg-name nsg-web \
  --name Allow-SSH \
  --priority 100 \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

az network nsg rule create \
  --resource-group rg-devops-demo \
  --nsg-name nsg-web \
  --name Allow-HTTP \
  --priority 200 \
  --destination-port-ranges 80 \
  --access Allow \
  --protocol Tcp

# 4. Create public IP
az network public-ip create \
  --resource-group rg-devops-demo \
  --name pip-web-01 \
  --allocation-method Static \
  --sku Standard

# 5. Create VM
az vm create \
  --resource-group rg-devops-demo \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --vnet-name vnet-devops \
  --subnet subnet-web \
  --nsg nsg-web \
  --public-ip-address pip-web-01 \
  --tags Role=WebServer Environment=Dev

# 6. Get VM details
az vm show \
  --resource-group rg-devops-demo \
  --name vm-web-01 \
  --output table

# 7. Get public IP
PUBLIC_IP=$(az vm list-ip-addresses \
  --resource-group rg-devops-demo \
  --name vm-web-01 \
  --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" \
  --output tsv)

echo "VM Public IP: $PUBLIC_IP"

# 8. SSH to VM
ssh azureuser@$PUBLIC_IP

# On VM: Install Nginx
sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx

# 9. Test from browser
curl http://$PUBLIC_IP

# 10. Stop VM
az vm stop --resource-group rg-devops-demo --name vm-web-01

# 11. Deallocate VM (stop billing)
az vm deallocate --resource-group rg-devops-demo --name vm-web-01

# 12. Start VM
az vm start --resource-group rg-devops-demo --name vm-web-01

# 13. Delete VM
az vm delete \
  --resource-group rg-devops-demo \
  --name vm-web-01 \
  --yes
```

**Challenge:**
Deploy 3 VMs in different availability zones with:
1. Load balancer in front
2. Shared storage
3. Auto-shutdown schedule
4. Monitoring enabled

---

### Exercise 3: Azure Storage Operations (45 minutes)

**Objective:** Work with Azure Storage

```bash
# 1. Create storage account
STORAGE_NAME="stdevops$RANDOM"
az storage account create \
  --name $STORAGE_NAME \
  --resource-group rg-devops-demo \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot

# 2. Get connection string
CONNECTION_STRING=$(az storage account show-connection-string \
  --name $STORAGE_NAME \
  --resource-group rg-devops-demo \
  --query connectionString \
  --output tsv)

# 3. Create blob container
az storage container create \
  --name logs \
  --connection-string $CONNECTION_STRING \
  --public-access off

az storage container create \
  --name backups \
  --connection-string $CONNECTION_STRING

# 4. Upload files
echo "Application log" > app.log
az storage blob upload \
  --container-name logs \
  --file app.log \
  --name "2024/01/app.log" \
  --connection-string $CONNECTION_STRING

# 5. List blobs
az storage blob list \
  --container-name logs \
  --connection-string $CONNECTION_STRING \
  --output table

# 6. Download blob
az storage blob download \
  --container-name logs \
  --name "2024/01/app.log" \
  --file downloaded.log \
  --connection-string $CONNECTION_STRING

# 7. Generate SAS token
END_DATE=$(date -u -d "1 hour" '+%Y-%m-%dT%H:%MZ')
SAS_TOKEN=$(az storage container generate-sas \
  --name logs \
  --account-name $STORAGE_NAME \
  --permissions rl \
  --expiry $END_DATE \
  --connection-string $CONNECTION_STRING \
  --output tsv)

# 8. Create lifecycle policy
cat > lifecycle.json << 'EOF'
{
  "rules": [
    {
      "enabled": true,
      "name": "move-to-cool",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        }
      }
    }
  ]
}
EOF

az storage account management-policy create \
  --account-name $STORAGE_NAME \
  --policy @lifecycle.json \
  --resource-group rg-devops-demo

# 9. Enable versioning
az storage account blob-service-properties update \
  --account-name $STORAGE_NAME \
  --resource-group rg-devops-demo \
  --enable-versioning true

# 10. Enable soft delete
az storage account blob-service-properties update \
  --account-name $STORAGE_NAME \
  --resource-group rg-devops-demo \
  --enable-delete-retention true \
  --delete-retention-days 7
```

**Challenge:**
Create a backup solution that:
1. Backs up to blob storage daily
2. Uses lifecycle policies
3. Replicates to secondary region
4. Sends alerts on failures

---

### Exercise 4: Deploy AKS Cluster (90 minutes)

**Objective:** Deploy and manage AKS cluster

```bash
# 1. Create AKS cluster
az aks create \
  --resource-group rg-devops-demo \
  --name aks-devops-cluster \
  --node-count 2 \
  --node-vm-size Standard_B2s \
  --enable-managed-identity \
  --generate-ssh-keys \
  --network-plugin azure \
  --enable-addons monitoring

# 2. Get credentials
az aks get-credentials \
  --resource-group rg-devops-demo \
  --name aks-devops-cluster

# 3. Verify connection
kubectl get nodes

# 4. Create ACR
ACR_NAME="acrdevops$RANDOM"
az acr create \
  --resource-group rg-devops-demo \
  --name $ACR_NAME \
  --sku Basic

# 5. Attach ACR to AKS
az aks update \
  --resource-group rg-devops-demo \
  --name aks-devops-cluster \
  --attach-acr $ACR_NAME

# 6. Build and push image
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EOF

echo "<h1>Hello from AKS!</h1>" > index.html

az acr build \
  --registry $ACR_NAME \
  --image myapp:v1 \
  .

# 7. Deploy to AKS
cat > deployment.yaml << EOF
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
    spec:
      containers:
      - name: myapp
        image: ${ACR_NAME}.azurecr.io/myapp:v1
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: myapp
EOF

kubectl apply -f deployment.yaml

# 8. Wait for external IP
kubectl get service myapp --watch

# 9. Scale deployment
kubectl scale deployment myapp --replicas=5

# 10. View logs
kubectl logs -l app=myapp --tail=50

# 11. Create HPA
kubectl autoscale deployment myapp \
  --cpu-percent=50 \
  --min=3 \
  --max=10
```

**Challenge:**
Deploy a microservices app with:
1. Multiple services
2. Ingress controller
3. Cert-manager for TLS
4. Monitoring with Prometheus

---

### Exercise 5: Azure Key Vault (45 minutes)

**Objective:** Secure secrets with Key Vault

```bash
# 1. Create Key Vault
KV_NAME="kv-devops-$RANDOM"
az keyvault create \
  --name $KV_NAME \
  --resource-group rg-devops-demo \
  --location eastus

# 2. Add secrets
az keyvault secret set \
  --vault-name $KV_NAME \
  --name "DatabasePassword" \
  --value "P@ssw0rd123!"

az keyvault secret set \
  --vault-name $KV_NAME \
  --name "ApiKey" \
  --value "sk-1234567890"

# 3. List secrets
az keyvault secret list \
  --vault-name $KV_NAME \
  --output table

# 4. Get secret value
az keyvault secret show \
  --vault-name $KV_NAME \
  --name "DatabasePassword" \
  --query value \
  --output tsv

# 5. Create VM with managed identity
az vm create \
  --resource-group rg-devops-demo \
  --name vm-app-01 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --assign-identity

# 6. Get VM identity
VM_IDENTITY=$(az vm show \
  --resource-group rg-devops-demo \
  --name vm-app-01 \
  --query identity.principalId \
  --output tsv)

# 7. Grant Key Vault access
az keyvault set-policy \
  --name $KV_NAME \
  --object-id $VM_IDENTITY \
  --secret-permissions get list

# 8. Access from VM
# SSH to VM and run:
az login --identity
az keyvault secret show \
  --vault-name $KV_NAME \
  --name "DatabasePassword" \
  --query value \
  --output tsv
```

---

### Exercise 6: Azure DevOps Pipeline (60 minutes)

**Objective:** Create CI/CD pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  azureSubscription: 'MyAzureConnection'
  resourceGroup: 'rg-devops-demo'
  acrName: 'acrdevops'
  aksCluster: 'aks-devops-cluster'

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - task: Docker@2
      displayName: 'Build Docker image'
      inputs:
        command: build
        dockerfile: '**/Dockerfile'
        tags: |
          $(Build.BuildId)
          latest
    
    - task: Docker@2
      displayName: 'Push to ACR'
      inputs:
        command: push
        containerRegistry: $(azureSubscription)
        repository: myapp
        tags: |
          $(Build.BuildId)
          latest

- stage: Deploy
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployToAKS
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            displayName: 'Deploy to AKS'
            inputs:
              action: deploy
              kubernetesServiceConnection: $(aksCluster)
              manifests: |
                k8s/deployment.yaml
                k8s/service.yaml
              containers: |
                $(acrName).azurecr.io/myapp:$(Build.BuildId)
```

---

### Exercise 7: Monitoring with Azure Monitor (45 minutes)

**Objective:** Set up comprehensive monitoring

```bash
# 1. Create Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group rg-devops-demo \
  --workspace-name law-devops

# 2. Get workspace ID
WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group rg-devops-demo \
  --workspace-name law-devops \
  --query customerId \
  --output tsv)

# 3. Create Application Insights
az monitor app-insights component create \
  --app myapp-insights \
  --location eastus \
  --resource-group rg-devops-demo \
  --workspace $WORKSPACE_ID

# 4. Create action group
az monitor action-group create \
  --resource-group rg-devops-demo \
  --name ag-devops-alerts \
  --short-name DevOps \
  --email-receiver name=admin email=admin@example.com

# 5. Create metric alert
az monitor metrics alert create \
  --name alert-high-cpu \
  --resource-group rg-devops-demo \
  --scopes /subscriptions/.../resourceGroups/rg-devops-demo/providers/Microsoft.Compute/virtualMachines/vm-web-01 \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action ag-devops-alerts

# 6. Query logs
az monitor log-analytics query \
  --workspace $WORKSPACE_ID \
  --analytics-query "AzureActivity | where TimeGenerated > ago(1h) | summarize count() by OperationName" \
  --output table
```

---

## 🎯 Practice Projects

### Project 1: Three-Tier Web Application
Deploy complete web app:
- Application Gateway in public subnet
- VMs in private subnet
- Azure SQL Database
- VM Scale Set with auto-scaling
- Azure Monitor for monitoring
- Blob Storage for static assets
- Azure CDN for global delivery

### Project 2: Microservices on AKS
Build microservices platform:
- AKS cluster with multiple node pools
- Azure Container Registry
- Ingress controller with TLS
- Azure Service Bus for messaging
- Cosmos DB for data
- Azure Monitor for containers
- GitOps with Flux

### Project 3: CI/CD Pipeline
Create complete CI/CD:
- Azure Repos for source control
- Azure Pipelines for build/deploy
- Multiple environments (dev/qa/prod)
- Approval gates
- Automated testing
- Security scanning
- Rollback capability

### Project 4: Hybrid Cloud Setup
Implement hybrid solution:
- Azure Arc for on-premises servers
- Azure Stack for edge computing
- ExpressRoute for connectivity
- Azure Site Recovery for DR
- Hybrid identity with Azure AD
- Centralized monitoring

---

## 📝 Best Practices

### 1. Security
- Enable Azure AD authentication
- Use RBAC with least privilege
- Implement Managed Identities
- Enable Azure Security Center
- Use Private Endpoints
- Enable encryption at rest
- Regular security audits

### 2. Cost Optimization
- Use Azure Cost Management
- Implement auto-shutdown for dev/test
- Right-size resources
- Use Reserved Instances
- Enable Azure Advisor
- Set up budget alerts
- Delete unused resources

### 3. High Availability
- Deploy across Availability Zones
- Use Azure Site Recovery
- Implement geo-redundant storage
- Configure auto-scaling
- Use Azure Traffic Manager
- Regular DR testing

### 4. Monitoring
- Enable Azure Monitor
- Set up Application Insights
- Create custom dashboards
- Configure alerts
- Use Log Analytics
- Implement distributed tracing

---

## ✅ Completion Checklist

### Week 1
- [ ] Set up Azure account
- [ ] Configure Azure CLI
- [ ] Create resource groups
- [ ] Deploy VMs
- [ ] Work with storage accounts

### Week 2
- [ ] Create VNets and subnets
- [ ] Configure NSGs
- [ ] Deploy Load Balancer
- [ ] Set up VPN Gateway
- [ ] Understand networking

### Week 3
- [ ] Work with Blob Storage
- [ ] Create Azure SQL Database
- [ ] Use Cosmos DB
- [ ] Configure Redis Cache
- [ ] Implement backups

### Week 4
- [ ] Configure Azure AD
- [ ] Implement RBAC
- [ ] Use Key Vault
- [ ] Enable Security Center
- [ ] Create Managed Identities

### Week 5
- [ ] Deploy AKS cluster
- [ ] Use Azure Container Registry
- [ ] Deploy applications to AKS
- [ ] Configure Helm
- [ ] Implement GitOps

### Week 6
- [ ] Create Azure Pipelines
- [ ] Build CI/CD workflows
- [ ] Use ARM templates
- [ ] Implement Terraform
- [ ] Complete projects

---

**Next:** [GCP →](../gcp/README.md)