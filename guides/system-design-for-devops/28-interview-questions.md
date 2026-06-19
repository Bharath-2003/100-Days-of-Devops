# Common System Design Interview Questions for DevOps

## 📋 Overview

This guide covers the most common system design questions asked in DevOps, Platform Engineer, and SRE interviews, with detailed solutions and approaches.

---

## 🎯 Question Categories

1. **CI/CD Pipeline Design**
2. **Infrastructure Architecture**
3. **Monitoring & Observability**
4. **High Availability & Disaster Recovery**
5. **Scalability & Performance**
6. **Security & Compliance**
7. **Cost Optimization**

---

## 1. CI/CD Pipeline Design Questions

### Q1: Design a CI/CD Pipeline for a Microservices Application

**Difficulty:** Medium-Hard

**Requirements:**
- 20 microservices
- Multiple teams working independently
- Deploy to Kubernetes
- Zero-downtime deployments
- Automated testing at multiple levels
- Security scanning
- Rollback capability

**Solution Approach:**

#### High-Level Architecture
```
[Git Repository]
    ↓
[Webhook Trigger]
    ↓
[CI Pipeline]
├─ Code Checkout
├─ Unit Tests
├─ Code Quality (SonarQube)
├─ Security Scan (Trivy/Snyk)
├─ Build Docker Image
├─ Push to Registry
└─ Integration Tests
    ↓
[CD Pipeline]
├─ Deploy to Dev
├─ Automated Tests
├─ Deploy to Staging
├─ Performance Tests
├─ Security Tests
└─ Deploy to Production
    ├─ Canary Deployment
    ├─ Health Checks
    └─ Full Rollout
```

#### Detailed Design

**1. Source Control & Branching Strategy**
```
GitFlow Model:
- main: Production code
- develop: Integration branch
- feature/*: Feature branches
- release/*: Release candidates
- hotfix/*: Emergency fixes

Monorepo vs Polyrepo:
- Monorepo: Single repo, path-based triggers
- Polyrepo: Separate repos per service
```

**2. CI Pipeline (Per Service)**
```yaml
# .gitlab-ci.yml or Jenkinsfile
stages:
  - validate
  - test
  - build
  - scan
  - publish

validate:
  - Lint code
  - Check formatting
  - Validate configs

test:
  - Unit tests (coverage > 80%)
  - Integration tests
  - Contract tests

build:
  - Build application
  - Build Docker image
  - Tag with commit SHA + semantic version

scan:
  - Security scan (Trivy)
  - Dependency check (Snyk)
  - License compliance

publish:
  - Push to Harbor/ECR
  - Update Helm chart
  - Trigger CD pipeline
```

**3. CD Pipeline (GitOps with ArgoCD)**
```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: service-a
spec:
  project: default
  source:
    repoURL: https://github.com/org/helm-charts
    targetRevision: HEAD
    path: services/service-a
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

**4. Deployment Strategy**
```
Canary Deployment:
1. Deploy 10% traffic to new version
2. Monitor metrics for 10 minutes
3. If healthy, increase to 50%
4. Monitor for 10 minutes
5. If healthy, complete rollout to 100%
6. If unhealthy at any stage, automatic rollback

Tools:
- Flagger (for automated canary)
- Istio/Linkerd (for traffic splitting)
- Prometheus (for metrics)
```

**5. Testing Strategy**
```
Test Pyramid:
├─ Unit Tests (70%)
│  - Run in CI
│  - Fast feedback
│  - Mock dependencies
│
├─ Integration Tests (20%)
│  - Test service interactions
│  - Use test containers
│  - Run in CI
│
└─ E2E Tests (10%)
   - Full user flows
   - Run in staging
   - Smoke tests in production
```

**6. Monitoring & Rollback**
```
Metrics to Monitor:
- Error rate
- Response time (p50, p95, p99)
- Request rate
- CPU/Memory usage
- Custom business metrics

Automatic Rollback Triggers:
- Error rate > 5%
- p99 latency > 1s
- Failed health checks
- Manual trigger

Rollback Process:
1. Detect issue (automated or manual)
2. Stop deployment
3. Route traffic to previous version
4. Investigate and fix
5. Redeploy when ready
```

**Key Technologies:**
- **CI:** GitLab CI, Jenkins, GitHub Actions
- **CD:** ArgoCD, Flux, Spinnaker
- **Container Registry:** Harbor, ECR, GCR
- **Orchestration:** Kubernetes
- **Service Mesh:** Istio, Linkerd
- **Monitoring:** Prometheus, Grafana, Datadog

**Trade-offs:**
- ✅ Automated, fast deployments
- ✅ Independent team workflows
- ✅ Safety with canary deployments
- ❌ Complex setup and maintenance
- ❌ Requires skilled team
- ❌ Higher infrastructure costs

---

### Q2: Design a Multi-Environment CI/CD Pipeline

**Difficulty:** Medium

**Requirements:**
- Dev, Staging, Production environments
- Different approval processes
- Environment-specific configurations
- Secrets management
- Audit trail

**Solution:**

```
[Git Push]
    ↓
[CI Pipeline] → Build & Test
    ↓
[Deploy to Dev] → Automatic
    ↓
[Automated Tests]
    ↓
[Deploy to Staging] → Automatic after tests pass
    ↓
[Manual Approval] → QA Team
    ↓
[Deploy to Production] → Manual approval required
    ↓
[Smoke Tests]
    ↓
[Monitoring]
```

**Environment Configuration:**
```yaml
# Using Helm values per environment
dev-values.yaml:
  replicas: 1
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
  ingress:
    host: dev.example.com

staging-values.yaml:
  replicas: 2
  resources:
    limits:
      cpu: 1000m
      memory: 1Gi
  ingress:
    host: staging.example.com

prod-values.yaml:
  replicas: 5
  resources:
    limits:
      cpu: 2000m
      memory: 2Gi
  ingress:
    host: www.example.com
  autoscaling:
    enabled: true
    minReplicas: 5
    maxReplicas: 20
```

**Secrets Management:**
```
Options:
1. HashiCorp Vault
   - Centralized secrets
   - Dynamic secrets
   - Audit logging

2. AWS Secrets Manager
   - Cloud-native
   - Automatic rotation
   - IAM integration

3. Sealed Secrets (Kubernetes)
   - GitOps friendly
   - Encrypted in Git
   - Decrypted in cluster

Implementation:
- Never commit secrets to Git
- Use environment variables
- Rotate secrets regularly
- Audit access logs
```

---

## 2. Infrastructure Architecture Questions

### Q3: Design a Highly Available Web Application

**Difficulty:** Medium

**Requirements:**
- 99.99% availability (52 minutes downtime/year)
- Handle 10,000 requests/second
- Global user base
- <200ms response time
- Auto-scaling capability

**Solution:**

#### Multi-Region Architecture
```
[Route53 - DNS]
├─ Health Checks
└─ Geo-routing
    ↓
[CloudFront CDN]
├─ Edge Locations
└─ Static Content
    ↓
[Region 1 (Primary)]          [Region 2 (DR)]
├─ ALB (Multi-AZ)             ├─ ALB (Multi-AZ)
├─ Auto Scaling Group         ├─ Auto Scaling Group
│  ├─ AZ-1a: 3 instances     │  ├─ AZ-2a: 2 instances
│  ├─ AZ-1b: 3 instances     │  ├─ AZ-2b: 2 instances
│  └─ AZ-1c: 3 instances     │  └─ AZ-2c: 2 instances
├─ ElastiCache (Redis)        ├─ ElastiCache (Redis)
└─ RDS (Multi-AZ)            └─ RDS (Read Replica)
    ├─ Master                     └─ Async Replication
    └─ Standby
```

#### Component Details

**1. DNS & CDN Layer**
```
Route53:
- Health checks every 30 seconds
- Failover to secondary region if primary fails
- Geo-routing for optimal latency
- TTL: 60 seconds for fast failover

CloudFront:
- Cache static assets (images, CSS, JS)
- Edge locations worldwide
- SSL/TLS termination
- DDoS protection
```

**2. Load Balancing**
```
Application Load Balancer:
- Multi-AZ deployment
- Health checks: /health endpoint
- Connection draining: 300 seconds
- Sticky sessions if needed
- SSL termination

Configuration:
- Target groups per service
- Path-based routing
- Host-based routing
- WebSocket support
```

**3. Application Tier**
```
Auto Scaling Group:
- Min: 6 instances (2 per AZ)
- Max: 30 instances
- Desired: 9 instances (3 per AZ)

Scaling Policies:
- Scale out: CPU > 70% for 2 minutes
- Scale in: CPU < 30% for 5 minutes
- Target tracking: 60% CPU utilization

Health Checks:
- ELB health check
- EC2 status check
- Custom application health check
```

**4. Caching Layer**
```
ElastiCache (Redis):
- Cluster mode enabled
- Multi-AZ with automatic failover
- 3 shards, 2 replicas per shard
- Backup enabled (daily)

Cache Strategy:
- Session data: TTL 30 minutes
- User profiles: TTL 1 hour
- Product catalog: TTL 5 minutes
- Cache-aside pattern
```

**5. Database Layer**
```
RDS (PostgreSQL):
Primary Region:
- Multi-AZ deployment
- Master in AZ-1a
- Standby in AZ-1b
- Automatic failover < 2 minutes
- Automated backups (7 days retention)
- Point-in-time recovery

Secondary Region:
- Read replica
- Async replication
- Can be promoted to master
- Used for disaster recovery

Connection Pooling:
- PgBouncer or RDS Proxy
- Max connections: 1000
- Pool size: 100 per instance
```

**6. Monitoring & Alerting**
```
CloudWatch Metrics:
- ALB: Request count, latency, errors
- EC2: CPU, memory, disk, network
- RDS: Connections, CPU, IOPS
- ElastiCache: CPU, memory, evictions

Alarms:
- Critical: Page on-call
  - ALB 5xx errors > 1%
  - RDS CPU > 90%
  - Any instance unhealthy
  
- Warning: Email team
  - ALB latency > 500ms
  - RDS connections > 80%
  - Cache hit rate < 80%

Dashboards:
- Real-time metrics
- SLA tracking
- Cost monitoring
```

**Availability Calculation:**
```
Component Availability:
- Route53: 100%
- CloudFront: 99.99%
- ALB: 99.99%
- EC2 (Multi-AZ): 99.99%
- RDS (Multi-AZ): 99.95%
- ElastiCache: 99.99%

Overall: 99.99% × 99.99% × 99.99% × 99.95% × 99.99%
       ≈ 99.96% (within 99.99% target with redundancy)
```

**Cost Optimization:**
- Reserved Instances for baseline capacity
- Spot Instances for burst capacity
- S3 Intelligent-Tiering for backups
- CloudFront for reduced data transfer
- Right-sizing based on metrics

---

### Q4: Design a Kubernetes Cluster for Production

**Difficulty:** Hard

**Requirements:**
- Multi-tenant (3 teams)
- High availability
- Auto-scaling (nodes and pods)
- Network policies
- Monitoring and logging
- Disaster recovery

**Solution:**

#### Cluster Architecture
```
[Control Plane] (Managed - EKS/GKE/AKS)
├─ API Server (HA)
├─ etcd (HA, backed up)
├─ Scheduler
└─ Controller Manager
    ↓
[Worker Nodes]
├─ Node Pool 1: General workloads
│  ├─ Min: 3 nodes
│  ├─ Max: 20 nodes
│  └─ Instance type: t3.xlarge
│
├─ Node Pool 2: Memory-intensive
│  ├─ Min: 2 nodes
│  ├─ Max: 10 nodes
│  └─ Instance type: r5.2xlarge
│
└─ Node Pool 3: GPU workloads
   ├─ Min: 0 nodes
   ├─ Max: 5 nodes
   └─ Instance type: p3.2xlarge
```

#### Multi-Tenancy Setup
```yaml
# Namespace per team
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
  labels:
    team: team-a
---
# Resource Quotas
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "100"
    requests.memory: 200Gi
    limits.cpu: "200"
    limits.memory: 400Gi
    pods: "100"
---
# Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: team-a-network-policy
  namespace: team-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: team-a
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          team: team-a
  - to:  # Allow external traffic
    - namespaceSelector: {}
```

#### Auto-Scaling Configuration
```yaml
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 3
  maxReplicas: 50
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
---
# Cluster Autoscaler
# Automatically scales node pools based on pending pods
# Configuration via cloud provider (EKS/GKE/AKS)
```

#### Monitoring Stack
```yaml
# Prometheus Operator
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
spec:
  replicas: 2
  retention: 30d
  storage:
    volumeClaimTemplate:
      spec:
        resources:
          requests:
            storage: 100Gi
  serviceMonitorSelector:
    matchLabels:
      monitoring: enabled
---
# Grafana
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        volumeMounts:
        - name: dashboards
          mountPath: /var/lib/grafana/dashboards
```

#### Logging Stack
```yaml
# Fluent Bit DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
spec:
  template:
    spec:
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

#### Disaster Recovery
```
Backup Strategy:
1. etcd backups (every 6 hours)
2. Persistent volume snapshots (daily)
3. Configuration backups (Git)
4. Secrets backup (encrypted)

Recovery Plan:
1. Restore etcd from backup
2. Recreate cluster from IaC
3. Restore PVs from snapshots
4. Redeploy applications via GitOps
5. Verify functionality

RTO: 4 hours
RPO: 6 hours
```

---

## 3. Monitoring & Observability Questions

### Q5: Design a Monitoring and Alerting System

**Difficulty:** Medium

**Requirements:**
- Monitor 500+ servers
- Microservices architecture
- Real-time alerting
- Historical data (1 year)
- Custom dashboards
- On-call rotation

**Solution:**

#### Architecture
```
[Applications]
    ↓
[Metrics Collection]
├─ Prometheus (Pull)
├─ StatsD (Push)
└─ OpenTelemetry
    ↓
[Time Series Database]
├─ Prometheus (short-term: 30 days)
└─ Thanos/Cortex (long-term: 1 year)
    ↓
[Visualization]
├─ Grafana Dashboards
└─ Custom UI
    ↓
[Alerting]
├─ Alertmanager
├─ PagerDuty
└─ Slack/Email
```

#### Metrics Collection
```yaml
# Prometheus scrape config
scrape_configs:
  # Application metrics
  - job_name: 'applications'
    kubernetes_sd_configs:
    - role: pod
    relabel_configs:
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
      action: keep
      regex: true
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
      action: replace
      target_label: __metrics_path__
      regex: (.+)
    - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
      action: replace
      regex: ([^:]+)(?::\d+)?;(\d+)
      replacement: $1:$2
      target_label: __address__
  
  # Node metrics
  - job_name: 'node-exporter'
    static_configs:
    - targets: ['node-exporter:9100']
  
  # Kubernetes metrics
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
    - role: endpoints
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

#### Alert Rules
```yaml
# prometheus-rules.yaml
groups:
- name: application_alerts
  interval: 30s
  rules:
  # High error rate
  - alert: HighErrorRate
    expr: |
      rate(http_requests_total{status=~"5.."}[5m])
      / rate(http_requests_total[5m]) > 0.05
    for: 5m
    labels:
      severity: critical
      team: backend
    annotations:
      summary: "High error rate on {{ $labels.instance }}"
      description: "Error rate is {{ $value | humanizePercentage }}"
  
  # High latency
  - alert: HighLatency
    expr: |
      histogram_quantile(0.99,
        rate(http_request_duration_seconds_bucket[5m])
      ) > 1
    for: 10m
    labels:
      severity: warning
      team: backend
    annotations:
      summary: "High latency on {{ $labels.instance }}"
      description: "P99 latency is {{ $value }}s"
  
  # Pod down
  - alert: PodDown
    expr: up{job="kubernetes-pods"} == 0
    for: 5m
    labels:
      severity: critical
      team: platform
    annotations:
      summary: "Pod {{ $labels.pod }} is down"
      description: "Pod has been down for more than 5 minutes"
  
  # High CPU
  - alert: HighCPU
    expr: |
      100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 10m
    labels:
      severity: warning
      team: platform
    annotations:
      summary: "High CPU on {{ $labels.instance }}"
      description: "CPU usage is {{ $value }}%"
  
  # High memory
  - alert: HighMemory
    expr: |
      (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)
      / node_memory_MemTotal_bytes * 100 > 90
    for: 10m
    labels:
      severity: warning
      team: platform
    annotations:
      summary: "High memory on {{ $labels.instance }}"
      description: "Memory usage is {{ $value }}%"
  
  # Disk space
  - alert: DiskSpaceLow
    expr: |
      (node_filesystem_avail_bytes{mountpoint="/"}
      / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 10
    for: 5m
    labels:
      severity: critical
      team: platform
    annotations:
      summary: "Low disk space on {{ $labels.instance }}"
      description: "Only {{ $value }}% disk space remaining"
```

#### Alertmanager Configuration
```yaml
# alertmanager.yaml
global:
  resolve_timeout: 5m
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'
  slack_api_url: 'https://hooks.slack.com/services/XXX'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  routes:
  # Critical alerts to PagerDuty
  - match:
      severity: critical
    receiver: 'pagerduty'
    continue: true
  
  # Warning alerts to Slack
  - match:
      severity: warning
    receiver: 'slack'
  
  # Team-specific routing
  - match:
      team: backend
    receiver: 'backend-team'
  
  - match:
      team: platform
    receiver: 'platform-team'

receivers:
- name: 'default'
  email_configs:
  - to: 'devops@example.com'

- name: 'pagerduty'
  pagerduty_configs:
  - service_key: 'YOUR_SERVICE_KEY'
    description: '{{ .GroupLabels.alertname }}'

- name: 'slack'
  slack_configs:
  - channel: '#alerts'
    title: '{{ .GroupLabels.alertname }}'
    text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

- name: 'backend-team'
  slack_configs:
  - channel: '#backend-alerts'
  email_configs:
  - to: 'backend-team@example.com'

- name: 'platform-team'
  slack_configs:
  - channel: '#platform-alerts'
  email_configs:
  - to: 'platform-team@example.com'

inhibit_rules:
# Inhibit warning if critical is firing
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  equal: ['alertname', 'instance']
```

#### Grafana Dashboards
```
Key Dashboards:

1. Overview Dashboard
   - Total requests/sec
   - Error rate
   - Average latency
   - Active users
   - System health

2. Application Dashboard (per service)
   - Request rate
   - Error rate by endpoint
   - Latency percentiles (p50, p95, p99)
   - Database queries
   - Cache hit rate

3. Infrastructure Dashboard
   - CPU usage per node
   - Memory usage per node
   - Disk I/O
   - Network traffic
   - Pod status

4. Business Metrics Dashboard
   - User signups
   - Transactions
   - Revenue
   - Conversion rate
   - Custom KPIs

5. SLA Dashboard
   - Uptime percentage
   - Error budget remaining
   - Incident count
   - MTTR (Mean Time To Recovery)
```

#### On-Call Rotation
```
PagerDuty Setup:
1. Escalation Policy
   - Level 1: Primary on-call (immediate)
   - Level 2: Secondary on-call (after 15 min)
   - Level 3: Manager (after 30 min)

2. Schedule
   - Weekly rotation
   - Handoff on Monday 9 AM
   - Backup on-call always assigned

3. Incident Response
   - Acknowledge within 5 minutes
   - Initial response within 15 minutes
   - Status updates every 30 minutes
   - Post-mortem within 48 hours
```

---

## 4. High Availability & Disaster Recovery

### Q6: Design a Disaster Recovery Solution

**Difficulty:** Hard

**Requirements:**
- RPO: 1 hour (max data loss)
- RTO: 4 hours (max downtime)
- Multi-region setup
- Automated failover
- Regular DR drills

**Solution:**

#### Architecture
```
[Primary Region - US-East]
├─ Application Tier (Active)
├─ Database (Master)
├─ Object Storage (S3)
└─ Backups (Continuous)
    ↓ (Replication)
[Secondary Region - US-West]
├─ Application Tier (Standby)
├─ Database (Read Replica)
├─ Object Storage (S3 Cross-Region Replication)
└─ Backups (Replicated)
```

#### Database Replication
```
PostgreSQL Setup:
Primary (US-East):
- Master database
- Write operations
- Continuous WAL archiving to S3
- Point-in-time recovery enabled

Secondary (US-West):
- Streaming replication from primary
- Read-only replica
- Lag < 30 seconds
- Can be promoted to master

Failover Process:
1. Detect primary failure (health checks)
2. Promote replica to master
3. Update DNS to point to new master
4. Redirect application traffic
5. Verify data integrity

Automated with:
- Patroni (PostgreSQL HA)
- PgBouncer (connection pooling)
- HAProxy (load balancing)
```

#### Application Failover
```yaml
# Route53 Health Check
{
  "Type": "HTTPS",
  "ResourcePath": "/health",
  "FullyQualifiedDomainName": "api.example.com",
  "Port": 443,
  "RequestInterval": 30,
  "FailureThreshold": 3
}

# Route53 Failover Record
{
  "Name": "api.example.com",
  "Type": "A",
  "SetIdentifier": "Primary",
  "Failover": "PRIMARY",
  "AliasTarget": {
    "HostedZoneId": "Z1234567890ABC",
    "DNSName": "primary-alb.us-east-1.elb.amazonaws.com",
    "EvaluateTargetHealth": true
  }
}

{
  "Name": "api.example.com",
  "Type": "A",
  "SetIdentifier": "Secondary",
  "Failover": "SECONDARY",
  "AliasTarget": {
    "HostedZoneId": "Z0987654321XYZ",
    "DNSName": "secondary-alb.us-west-2.elb.amazonaws.com",
    "EvaluateTargetHealth": true
  }
}
```

#### Backup Strategy
```
1. Database Backups
   - Full backup: Daily at 2 AM
   - Incremental: Every 6 hours
   - WAL archiving: Continuous
   - Retention: 30 days
   - Cross-region replication: Yes

2. Application Data
   - S3 versioning: Enabled
   - Cross-region replication: Enabled
   - Lifecycle policies: 
     - Standard: 30 days
     - IA: 30-90 days
     - Glacier: 90+ days

3. Configuration Backups
   - Infrastructure as Code in Git
   - Secrets in Vault (replicated)
   - Kubernetes manifests in Git
   - Regular exports of cloud configs

4. Testing
   - Monthly DR drills
   - Quarterly full failover tests
   - Document and improve process
```

#### Monitoring & Alerting
```
Critical Metrics:
- Replication lag < 30 seconds
- Backup success rate = 100%
- Health check success rate > 99%
- Cross-region replication lag < 5 minutes

Alerts:
- Replication lag > 1 minute: Warning
- Replication lag > 5 minutes: Critical
- Backup failure: Critical
- Health check failure: Critical
- Cross-region replication failure: Critical
```

#### DR Runbook
```
Failover Procedure:

1. Detection (Automated)
   - Health checks fail for 3 consecutive checks
   - Alert sent to on-call engineer
   - Automated failover initiated

2. Validation (Manual)
   - Verify primary region is truly down
   - Check replication status
   - Confirm data consistency

3. Failover (Semi-Automated)
   - Promote read replica to master
   - Update DNS records
   - Scale up secondary region
   - Redirect traffic
   - Verify application functionality

4. Communication
   - Update status page
   - Notify stakeholders
   - Post in incident channel

5. Post-Failover
   - Monitor secondary region
   - Investigate primary failure
   - Plan failback when ready

6. Failback (When Primary Recovered)
   - Sync data from secondary to primary
   - Verify data integrity
   - Reverse DNS changes
   - Monitor for issues
   - Document lessons learned
```

---

## 5. Scalability & Performance

### Q7: Design an Auto-Scaling Solution

**Difficulty:** Medium

**Requirements:**
- Handle traffic spikes (10x normal)
- Cost-effective
- Fast scale-up (<2 minutes)
- Gradual scale-down
- Predictive scaling for known patterns

**Solution:**

#### Multi-Layer Scaling Strategy
```
1. Application Layer (Kubernetes HPA)
   - Scale pods based on CPU/Memory
   - Custom metrics (requests/pod)
   - Fast response to load changes

2. Node Layer (Cluster Autoscaler)
   - Scale nodes based on pending pods
   - Multiple node pools
   - Spot instances for cost savings

3. Database Layer (Read Replicas)
   - Scale read replicas for read-heavy workloads
   - Connection pooling
   - Query caching

4. CDN Layer (CloudFront)
   - Automatic scaling
   - Edge caching
   - Reduce origin load
```

#### Implementation
```yaml
# Horizontal Pod Autoscaler with custom metrics
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 10
  maxReplicas: 100
  metrics:
  # CPU-based scaling
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  
  # Memory-based scaling
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  
  # Custom metric: requests per pod
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
  
  # External metric: SQS queue depth
  - type: External
    external:
      metric:
        name: sqs_queue_depth
        selector:
          matchLabels:
            queue: "processing-queue"
      target:
        type: AverageValue
        averageValue: "30"
  
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 5
        periodSeconds: 60
      selectPolicy: Min
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 10
        periodSeconds: 15
      selectPolicy: Max
```

#### Predictive Scaling
```python
# AWS Predictive Scaling Policy
{
  "MetricSpecifications": [
    {
      "TargetValue": 70.0,
      "PredefinedMetricPairSpecification": {
        "PredefinedMetricType": "ASGCPUUtilization"
      }
    }
  ],
  "Mode": "ForecastAndScale",
  "SchedulingBufferTime": 600,
  "MaxCapacityBreachBehavior": "IncreaseMaxCapacity",
  "MaxCapacityBuffer": 10
}

# Schedule-based scaling for known patterns
# Example: Scale up before business hours
{
  "ScheduledActions": [
    {
      "ScheduledActionName": "scale-up-morning",
      "Schedule": "0 8 * * MON-FRI",
      "MinSize": 20,
      "MaxSize": 100,
      "DesiredCapacity": 30
    },
    {
      "ScheduledActionName": "scale-down-evening",
      "Schedule": "0 20 * * MON-FRI",
      "MinSize": 10,
      "MaxSize": 50,
      "DesiredCapacity": 15
    }
  ]
}
```

#### Cost Optimization
```
Node Pool Strategy:

1. On-Demand Pool (Baseline)
   - Min: 5 nodes
   - Max: 20 nodes
   - For critical workloads
   - Always available

2. Reserved Instances Pool
   - 10 nodes (1-year commitment)
   - For predictable baseline
   - 40% cost savings

3. Spot Instances Pool
   - Min: 0 nodes
   - Max: 50 nodes
   - For burst capacity
   - 70% cost savings
   - Graceful handling of interruptions

Spot Instance Handling:
- Multiple instance types
- Spread across AZs
- Pod Disruption Budgets
- Graceful shutdown (120s)
- Automatic rescheduling
```

---

## Quick Reference: Interview Tips

### Before the Interview
- [ ] Review common patterns
- [ ] Practice drawing diagrams
- [ ] Prepare questions to ask
- [ ] Know your resume projects deeply

### During the Interview
- [ ] Ask clarifying questions
- [ ] Start with high-level design
- [ ] Explain your thinking process
- [ ] Discuss trade-offs
- [ ] Consider failure scenarios
- [ ] Think about operations

### Common Mistakes to Avoid
- ❌ Jumping to solution without requirements
- ❌ Over-engineering for current scale
- ❌ Ignoring failure scenarios
- ❌ Not discussing trade-offs
- ❌ Using buzzwords without understanding
- ❌ Forgetting about monitoring

### What Interviewers Look For
- ✅ Problem-solving approach
- ✅ Technical depth
- ✅ Practical experience
- ✅ Communication skills
- ✅ Trade-off analysis
- ✅ Operational thinking

---

**Next:** [Case Studies →](29-case-studies.md)

**Last Updated:** June 19, 2026