# System Design Quick Reference for DevOps

## 🚀 5-Minute Cheat Sheet

### Essential Concepts

#### CAP Theorem
```
Pick 2 of 3:
├─ Consistency: All nodes see same data
├─ Availability: Every request gets response
└─ Partition Tolerance: Works despite network failures

Examples:
- Banking: CP (Consistency + Partition)
- Social Media: AP (Availability + Partition)
```

#### Availability Nines
```
99.9%   (3 nines) = 8.76 hours downtime/year
99.99%  (4 nines) = 52.56 minutes downtime/year
99.999% (5 nines) = 5.26 minutes downtime/year
```

#### Scaling Types
```
Vertical (Scale Up):
✅ Simple, no code changes
❌ Hardware limits, single point of failure

Horizontal (Scale Out):
✅ No limits, better fault tolerance
❌ Complexity, state management
```

---

## 🏗️ Common Architecture Patterns

### 1. Three-Tier Architecture
```
[Load Balancer]
    ↓
[Web Servers] (Stateless)
    ↓
[App Servers] (Business Logic)
    ↓
[Database] (Data Layer)
```

### 2. Microservices
```
[API Gateway]
    ↓
[Service Mesh]
    ↓
[Service A] [Service B] [Service C]
    ↓           ↓           ↓
[DB A]      [DB B]      [DB C]
```

### 3. Event-Driven
```
[Producer] → [Message Queue] → [Consumer]
                    ↓
              [Consumer 2]
```

---

## 🔧 Load Balancing Algorithms

```
Round Robin:        A → B → C → A → B → C
Least Connections:  Send to server with fewest connections
IP Hash:            Same client → same server
Weighted:           Based on server capacity
```

---

## 💾 Caching Strategies

```
Cache-Aside (Lazy Loading):
1. Check cache
2. If miss, load from DB
3. Store in cache
4. Return data

Write-Through:
1. Write to cache
2. Write to DB
3. Return success

Write-Behind:
1. Write to cache
2. Async write to DB
3. Return success (faster)
```

---

## 🗄️ Database Patterns

### Replication
```
Master-Slave:
[Master] ← writes
    ↓ (replication)
[Slave 1] [Slave 2] ← reads

Master-Master:
[Master 1] ⟷ [Master 2]
(both accept writes)
```

### Sharding
```
Horizontal Partitioning:
User 1-1000   → Shard 1
User 1001-2000 → Shard 2
User 2001-3000 → Shard 3

Shard Key: user_id, region, date
```

---

## 🚢 Deployment Strategies

### Blue-Green
```
Blue (Current) ← 100% traffic
Green (New) ← 0% traffic
↓ (switch)
Blue ← 0% traffic
Green ← 100% traffic
```

### Canary
```
Old Version ← 90% traffic
New Version ← 10% traffic
↓ (if healthy)
Old Version ← 50% traffic
New Version ← 50% traffic
↓ (if healthy)
Old Version ← 0% traffic
New Version ← 100% traffic
```

### Rolling Update
```
Instance 1: Old → New
Instance 2: Old → New
Instance 3: Old → New
(one at a time)
```

---

## 📊 Key Metrics

### Application Metrics
```
Golden Signals:
├─ Latency: Response time
├─ Traffic: Requests per second
├─ Errors: Error rate
└─ Saturation: Resource utilization

RED Method:
├─ Rate: Requests per second
├─ Errors: Error rate
└─ Duration: Response time

USE Method:
├─ Utilization: % time busy
├─ Saturation: Queue depth
└─ Errors: Error count
```

### Latency Targets
```
Database Query:  < 10ms
API Response:    < 100ms
Page Load:       < 1s
Real-time:       < 100ms
```

### Throughput Levels
```
Small:      100-1K RPS
Medium:     1K-10K RPS
Large:      10K-100K RPS
Very Large: 100K+ RPS
```

---

## 🔐 Security Layers

```
Defense in Depth:
├─ Network: Firewall, Security Groups
├─ Application: WAF, Rate Limiting
├─ Authentication: OAuth, OIDC
├─ Authorization: RBAC, ABAC
├─ Data: Encryption at rest/transit
└─ Monitoring: Audit logs, SIEM
```

---

## 🎯 CI/CD Pipeline Stages

```
[Source] → [Build] → [Test] → [Deploy]
    ↓         ↓         ↓         ↓
  Git     Compile   Unit     Dev Env
          Docker    Integ    Staging
          Artifact  E2E      Prod
```

---

## ☸️ Kubernetes Components

```
Control Plane:
├─ API Server: Entry point
├─ etcd: State storage
├─ Scheduler: Pod placement
└─ Controller Manager: Reconciliation

Worker Nodes:
├─ kubelet: Node agent
├─ kube-proxy: Network rules
└─ Container Runtime: Docker/containerd
```

---

## 📈 Auto-Scaling Triggers

```
Horizontal Pod Autoscaler:
├─ CPU > 70%
├─ Memory > 80%
├─ Custom metrics (RPS, queue depth)
└─ Scale: min → max replicas

Cluster Autoscaler:
├─ Pending pods
├─ Node utilization < 50%
└─ Scale: min → max nodes
```

---

## 🔍 Monitoring Stack

```
Metrics:
├─ Collection: Prometheus, StatsD
├─ Storage: Prometheus, Thanos
└─ Visualization: Grafana

Logs:
├─ Collection: Fluent Bit, Fluentd
├─ Storage: Elasticsearch, Loki
└─ Visualization: Kibana, Grafana

Traces:
├─ Collection: OpenTelemetry
├─ Storage: Jaeger, Zipkin
└─ Visualization: Jaeger UI
```

---

## 🚨 Alert Severity Levels

```
Critical (P1):
- Page on-call immediately
- Service down
- Data loss
- Security breach

High (P2):
- Notify team
- Degraded performance
- High error rate
- Approaching limits

Medium (P3):
- Email notification
- Non-critical issues
- Warnings

Low (P4):
- Log only
- Informational
```

---

## 💰 Cost Optimization

```
Compute:
├─ Reserved Instances (1-3 year)
├─ Spot Instances (70% savings)
├─ Right-sizing (match workload)
└─ Auto-scaling (scale down)

Storage:
├─ Lifecycle policies
├─ Compression
├─ Deduplication
└─ Tiering (S3 IA, Glacier)

Network:
├─ CDN for static content
├─ VPC endpoints
├─ Data transfer optimization
└─ Regional placement
```

---

## 🎤 Interview Framework

### 1. Requirements (5 min)
```
Functional:
- What features?
- What use cases?

Non-Functional:
- How many users?
- What traffic?
- What availability?
- What latency?
```

### 2. Calculations (5 min)
```
- Users: DAU, MAU
- Traffic: RPS, peak
- Storage: data size, growth
- Bandwidth: in/out
```

### 3. High-Level Design (10 min)
```
Draw:
- Client layer
- Load balancer
- Application layer
- Cache layer
- Database layer
- Message queue
```

### 4. Deep Dive (15 min)
```
Discuss:
- Technology choices
- Scaling strategies
- Failure handling
- Data consistency
- Security
```

### 5. Wrap Up (5 min)
```
Cover:
- Bottlenecks
- Monitoring
- Trade-offs
- Future improvements
```

---

## 🛠️ Essential Tools

### Infrastructure
```
IaC:          Terraform, CloudFormation
Config Mgmt:  Ansible, Chef, Puppet
Containers:   Docker, Podman
Orchestration: Kubernetes, ECS
```

### CI/CD
```
CI:           Jenkins, GitLab CI, GitHub Actions
CD:           ArgoCD, Flux, Spinnaker
Registry:     Harbor, ECR, GCR
Artifacts:    Nexus, Artifactory
```

### Monitoring
```
Metrics:      Prometheus, Datadog
Logs:         ELK, Loki, Splunk
Tracing:      Jaeger, Zipkin
APM:          New Relic, Datadog
```

### Networking
```
Load Balancer: Nginx, HAProxy, ALB
Service Mesh:  Istio, Linkerd, Consul
DNS:          Route53, CoreDNS
CDN:          CloudFront, Cloudflare
```

---

## 📝 Common Mistakes

### ❌ Don't Do
- Jump to solution without requirements
- Over-engineer for current scale
- Ignore failure scenarios
- Forget monitoring
- Use buzzwords without understanding
- Design without considering cost

### ✅ Do Instead
- Ask clarifying questions
- Design for 10x current scale
- Plan for failures explicitly
- Include observability from start
- Explain trade-offs
- Consider operational costs

---

## 🎯 Key Takeaways

```
1. Always start with requirements
2. Think about scale and growth
3. Design for failure
4. Monitor everything
5. Automate operations
6. Consider costs
7. Document decisions
8. Practice regularly
```

---

## 📚 Study Checklist

### Fundamentals
- [ ] CAP theorem
- [ ] Consistency models
- [ ] Load balancing
- [ ] Caching strategies
- [ ] Database patterns

### DevOps Specific
- [ ] CI/CD pipelines
- [ ] Kubernetes architecture
- [ ] IaC patterns
- [ ] Deployment strategies
- [ ] GitOps workflows

### Reliability
- [ ] High availability
- [ ] Disaster recovery
- [ ] Auto-scaling
- [ ] Monitoring
- [ ] Incident response

### Security
- [ ] Authentication/Authorization
- [ ] Secrets management
- [ ] Network security
- [ ] Compliance
- [ ] Security scanning

### Practice
- [ ] Draw architecture diagrams
- [ ] Solve interview questions
- [ ] Review case studies
- [ ] Build sample projects
- [ ] Conduct mock interviews

---

## 🔗 Quick Links

- [Full Guide](README.md)
- [Introduction](01-introduction.md)
- [Interview Questions](28-interview-questions.md)

---

**Last Updated:** June 19, 2026
**Print this for quick reference during study sessions!**