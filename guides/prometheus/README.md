# Prometheus Monitoring for DevOps

## 🎯 Overview

Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability. It's the de facto standard for monitoring cloud-native applications and is a graduated CNCF project. This guide covers everything from basic metrics collection to advanced monitoring strategies.

**Duration:** 1.5 weeks
**Time Investment:** 4-5 hours/day
**Difficulty:** Intermediate
**Prerequisites:** Basic Linux, Docker, Kubernetes knowledge

---

## 📚 What You'll Learn

### Week 1: Prometheus Fundamentals
- Prometheus architecture
- Metrics types and data model
- PromQL query language
- Service discovery
- Exporters and instrumentation

### Week 2: Advanced Topics
- Alerting with Alertmanager
- High availability setup
- Long-term storage
- Federation
- Best practices

---

## 🎓 Important Concepts

### 1. Prometheus Architecture

```
Prometheus Architecture
┌─────────────────────────────────────────────────────┐
│                  Prometheus Server                  │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐│
│  │  Retrieval   │  │   Storage    │  │   HTTP    ││
│  │   (Scrape)   │→ │   (TSDB)     │→ │  Server   ││
│  └──────────────┘  └──────────────┘  └───────────┘│
│         ↑                                    ↓      │
└─────────┼────────────────────────────────────┼─────┘
          │                                    │
    ┌─────┴─────┐                        ┌────┴────┐
    │  Targets  │                        │ Queries │
    │           │                        │         │
    │ • Apps    │                        │ • API   │
    │ • Nodes   │                        │ • UI    │
    │ • Services│                        │ • Grafana│
    └───────────┘                        └─────────┘
          ↑
    ┌─────┴─────────┐
    │ Service       │
    │ Discovery     │
    │ • Kubernetes  │
    │ • Consul      │
    │ • File        │
    └───────────────┘

Components:
├─ Prometheus Server
│  ├─ Scrapes metrics from targets
│  ├─ Stores time-series data
│  └─ Evaluates alerting rules
│
├─ Exporters
│  ├─ Node Exporter (system metrics)
│  ├─ Blackbox Exporter (probing)
│  └─ Custom exporters
│
├─ Pushgateway
│  └─ For short-lived jobs
│
├─ Alertmanager
│  ├─ Handles alerts
│  ├─ Deduplication
│  └─ Routing and silencing
│
└─ Client Libraries
   └─ Instrument applications
```

---

### 2. Metrics Types

```
Prometheus Metric Types
├─ Counter
│  ├─ Monotonically increasing value
│  ├─ Resets to 0 on restart
│  └─ Examples: requests_total, errors_total
│
├─ Gauge
│  ├─ Value that can go up or down
│  ├─ Current state snapshot
│  └─ Examples: memory_usage, temperature
│
├─ Histogram
│  ├─ Samples observations
│  ├─ Counts in configurable buckets
│  └─ Examples: request_duration, response_size
│
└─ Summary
   ├─ Similar to histogram
   ├─ Calculates quantiles
   └─ Examples: request_duration_quantile
```

**Metric Naming Convention:**
```
<namespace>_<subsystem>_<name>_<unit>

Examples:
http_requests_total
http_request_duration_seconds
node_memory_usage_bytes
api_errors_total
database_connections_active
```

**Example Metrics:**
```python
from prometheus_client import Counter, Gauge, Histogram, Summary

# Counter - monotonically increasing
requests_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# Gauge - can go up or down
active_connections = Gauge(
    'active_connections',
    'Number of active connections'
)

# Histogram - observations in buckets
request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    buckets=[0.1, 0.5, 1.0, 2.0, 5.0]
)

# Summary - quantiles
request_latency = Summary(
    'http_request_latency_seconds',
    'HTTP request latency'
)

# Usage
requests_total.labels(method='GET', endpoint='/api', status='200').inc()
active_connections.set(42)
request_duration.observe(0.5)
request_latency.observe(0.3)
```

---

### 3. PromQL (Prometheus Query Language)

```
PromQL Query Types
├─ Instant Vector
│  └─ Set of time series at single timestamp
│
├─ Range Vector
│  └─ Set of time series over time range
│
├─ Scalar
│  └─ Simple numeric value
│
└─ String
   └─ String value (limited use)

Common Operators:
├─ Arithmetic: +, -, *, /, %, ^
├─ Comparison: ==, !=, >, <, >=, <=
├─ Logical: and, or, unless
└─ Aggregation: sum, avg, min, max, count
```

**Basic Queries:**
```promql
# Select all time series with metric name
http_requests_total

# Filter by labels
http_requests_total{method="GET"}
http_requests_total{method="GET", status="200"}

# Regular expressions
http_requests_total{endpoint=~"/api/.*"}
http_requests_total{status!~"5.."}

# Range vector (last 5 minutes)
http_requests_total[5m]

# Rate of increase (per second)
rate(http_requests_total[5m])

# Increase over time range
increase(http_requests_total[1h])

# Instant rate (irate)
irate(http_requests_total[5m])
```

**Aggregation Queries:**
```promql
# Sum across all instances
sum(http_requests_total)

# Sum by label
sum by (method) (http_requests_total)

# Average
avg(http_request_duration_seconds)

# Count
count(up == 1)

# Top 5 by value
topk(5, http_requests_total)

# Bottom 5 by value
bottomk(5, http_requests_total)

# Quantile
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

**Advanced Queries:**
```promql
# Request rate per second
sum(rate(http_requests_total[5m])) by (method)

# Error rate percentage
sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m])) * 100

# Average response time
rate(http_request_duration_seconds_sum[5m])
/
rate(http_request_duration_seconds_count[5m])

# Memory usage percentage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) 
/ 
node_memory_MemTotal_bytes * 100

# CPU usage percentage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Disk usage prediction (linear regression)
predict_linear(node_filesystem_avail_bytes[1h], 4 * 3600) < 0

# Alert if service down for 5 minutes
up == 0 and on(instance) up offset 5m == 0
```

---

### 4. Service Discovery

```
Service Discovery Methods
├─ Static Configuration
│  └─ Manual target list
│
├─ File-based
│  └─ JSON/YAML files
│
├─ Kubernetes
│  ├─ Pod discovery
│  ├─ Service discovery
│  └─ Ingress discovery
│
├─ Consul
│  └─ Service registry
│
├─ EC2
│  └─ AWS instances
│
└─ DNS
   └─ DNS SRV records
```

**Static Configuration:**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node-exporter'
    static_configs:
      - targets:
        - 'node1:9100'
        - 'node2:9100'
        - 'node3:9100'
        labels:
          env: 'production'
          region: 'us-east-1'
```

**File-based Discovery:**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'file-discovery'
    file_sd_configs:
      - files:
        - '/etc/prometheus/targets/*.json'
        - '/etc/prometheus/targets/*.yml'
        refresh_interval: 30s
```

```json
// /etc/prometheus/targets/web-servers.json
[
  {
    "targets": ["web1:9100", "web2:9100"],
    "labels": {
      "job": "web-servers",
      "env": "production"
    }
  }
]
```

**Kubernetes Discovery:**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      # Only scrape pods with annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      
      # Use custom port if specified
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      
      # Use custom path if specified
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      
      # Add namespace label
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      
      # Add pod name label
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name
```

---

### 5. Exporters

```
Common Exporters
├─ Node Exporter
│  ├─ CPU, memory, disk
│  ├─ Network statistics
│  └─ System metrics
│
├─ Blackbox Exporter
│  ├─ HTTP/HTTPS probing
│  ├─ TCP/ICMP checks
│  └─ DNS queries
│
├─ MySQL Exporter
│  └─ Database metrics
│
├─ PostgreSQL Exporter
│  └─ Database metrics
│
├─ Redis Exporter
│  └─ Cache metrics
│
├─ Nginx Exporter
│  └─ Web server metrics
│
└─ Custom Exporters
   └─ Application-specific
```

**Node Exporter Setup:**
```bash
# Download and install
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
sudo cp node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/

# Create systemd service
cat > /etc/systemd/system/node_exporter.service << 'EOF'
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/bin/node_exporter \
  --collector.filesystem.mount-points-exclude='^/(sys|proc|dev|host|etc)($$|/)' \
  --collector.netclass.ignored-devices='^(veth.*|docker.*|br-.*)$$'
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Start service
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter

# Verify
curl http://localhost:9100/metrics
```

**Blackbox Exporter Configuration:**
```yaml
# blackbox.yml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: []
      method: GET
      preferred_ip_protocol: "ip4"
  
  http_post_2xx:
    prober: http
    http:
      method: POST
      headers:
        Content-Type: application/json
      body: '{"key": "value"}'
  
  tcp_connect:
    prober: tcp
    timeout: 5s
  
  icmp:
    prober: icmp
    timeout: 5s
  
  dns:
    prober: dns
    dns:
      query_name: "example.com"
      query_type: "A"
```

**Custom Exporter (Python):**
```python
#!/usr/bin/env python3
from prometheus_client import start_http_server, Gauge, Counter
import time
import psutil

# Define metrics
cpu_usage = Gauge('custom_cpu_usage_percent', 'CPU usage percentage')
memory_usage = Gauge('custom_memory_usage_bytes', 'Memory usage in bytes')
disk_usage = Gauge('custom_disk_usage_percent', 'Disk usage percentage', ['mount_point'])
requests_total = Counter('custom_requests_total', 'Total requests', ['endpoint'])

def collect_metrics():
    """Collect system metrics"""
    while True:
        # CPU usage
        cpu_usage.set(psutil.cpu_percent(interval=1))
        
        # Memory usage
        memory = psutil.virtual_memory()
        memory_usage.set(memory.used)
        
        # Disk usage
        for partition in psutil.disk_partitions():
            try:
                usage = psutil.disk_usage(partition.mountpoint)
                disk_usage.labels(mount_point=partition.mountpoint).set(usage.percent)
            except PermissionError:
                continue
        
        time.sleep(15)

if __name__ == '__main__':
    # Start HTTP server
    start_http_server(9200)
    print("Custom exporter started on port 9200")
    
    # Collect metrics
    collect_metrics()
```

---

### 6. Alerting with Alertmanager

```
Alerting Flow
┌─────────────┐
│ Prometheus  │
│   Rules     │
└──────┬──────┘
       │ Alerts
       ▼
┌─────────────┐
│Alertmanager │
├─────────────┤
│ • Grouping  │
│ • Inhibition│
│ • Silencing │
│ • Routing   │
└──────┬──────┘
       │
       ├─► Email
       ├─► Slack
       ├─► PagerDuty
       └─► Webhook
```

**Alert Rules:**
```yaml
# /etc/prometheus/rules/alerts.yml
groups:
  - name: instance_alerts
    interval: 30s
    rules:
      # Instance down
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} has been down for more than 5 minutes."
      
      # High CPU usage
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"
      
      # High memory usage
      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}% on {{ $labels.instance }}"
      
      # Disk space low
      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Disk {{ $labels.mountpoint }} has only {{ $value }}% free space"
      
      # High error rate
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"
      
      # SSL certificate expiring
      - alert: SSLCertExpiringSoon
        expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 30
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "SSL certificate expiring soon"
          description: "SSL certificate for {{ $labels.instance }} expires in {{ $value | humanizeDuration }}"
```

**Alertmanager Configuration:**
```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@example.com'
  smtp_auth_username: 'alerts@example.com'
  smtp_auth_password: 'password'

# Templates
templates:
  - '/etc/alertmanager/templates/*.tmpl'

# Route tree
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
    
    # Database alerts to DBA team
    - match_re:
        service: ^(mysql|postgres|redis)$
      receiver: 'dba-team'
    
    # Infrastructure alerts
    - match:
        team: infrastructure
      receiver: 'infra-team'

# Inhibition rules
inhibit_rules:
  # Inhibit warning if critical alert firing
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']

# Receivers
receivers:
  - name: 'default'
    email_configs:
      - to: 'team@example.com'
        headers:
          Subject: '[ALERT] {{ .GroupLabels.alertname }}'
  
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
        description: '{{ .CommonAnnotations.summary }}'
  
  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#alerts'
        title: '{{ .CommonAnnotations.summary }}'
        text: '{{ .CommonAnnotations.description }}'
        send_resolved: true
  
  - name: 'dba-team'
    email_configs:
      - to: 'dba@example.com'
  
  - name: 'infra-team'
    email_configs:
      - to: 'infra@example.com'
```

---

### 7. High Availability Setup

```
HA Architecture
┌─────────────────────────────────────┐
│         Load Balancer               │
└────────┬────────────────────┬───────┘
         │                    │
    ┌────▼────┐          ┌────▼────┐
    │Prometheus│          │Prometheus│
    │ Server 1 │          │ Server 2 │
    └────┬────┘          └────┬────┘
         │                    │
         └────────┬───────────┘
                  │
         ┌────────▼────────┐
         │  Alertmanager   │
         │    Cluster      │
         │  ┌───┐ ┌───┐   │
         │  │AM1│ │AM2│   │
         │  └───┘ └───┘   │
         └─────────────────┘

HA Strategies:
├─ Multiple Prometheus servers
│  └─ Scrape same targets
│
├─ Alertmanager clustering
│  └─ Deduplicate alerts
│
├─ Remote storage
│  └─ Long-term retention
│
└─ Federation
   └─ Hierarchical setup
```

**Prometheus HA Configuration:**
```yaml
# prometheus-1.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    replica: 'prometheus-1'
    cluster: 'production'

alerting:
  alertmanagers:
    - static_configs:
      - targets:
        - 'alertmanager-1:9093'
        - 'alertmanager-2:9093'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node-exporter'
    static_configs:
      - targets:
        - 'node1:9100'
        - 'node2:9100'
```

**Alertmanager Cluster:**
```yaml
# alertmanager-1.yml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'

receivers:
  - name: 'default'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#alerts'

# Cluster configuration
cluster:
  listen-address: '0.0.0.0:9094'
  peers:
    - 'alertmanager-2:9094'
```

**Docker Compose HA Setup:**
```yaml
version: '3.8'

services:
  prometheus-1:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus-1.yml:/etc/prometheus/prometheus.yml
      - prometheus-1-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.external-url=http://prometheus-1:9090'
    ports:
      - "9090:9090"
  
  prometheus-2:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus-2.yml:/etc/prometheus/prometheus.yml
      - prometheus-2-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.external-url=http://prometheus-2:9090'
    ports:
      - "9091:9090"
  
  alertmanager-1:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--cluster.listen-address=0.0.0.0:9094'
      - '--cluster.peer=alertmanager-2:9094'
    ports:
      - "9093:9093"
  
  alertmanager-2:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--cluster.listen-address=0.0.0.0:9094'
      - '--cluster.peer=alertmanager-1:9094'
    ports:
      - "9094:9093"

volumes:
  prometheus-1-data:
  prometheus-2-data:
```

---

## 🔨 Hands-On Exercises

### Exercise 1: Deploy Prometheus Stack (60 minutes)

**Objective:** Set up complete monitoring stack

```bash
# Create directory structure
mkdir -p prometheus-stack/{prometheus,alertmanager,grafana}
cd prometheus-stack

# Prometheus configuration
cat > prometheus/prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
      - targets: ['alertmanager:9093']

rule_files:
  - '/etc/prometheus/rules/*.yml'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
  
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
EOF

# Alert rules
mkdir -p prometheus/rules
cat > prometheus/rules/alerts.yml << 'EOF'
groups:
  - name: basic_alerts
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
      
      - alert: HighCPU
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
EOF

# Alertmanager configuration
cat > alertmanager/alertmanager.yml << 'EOF'
global:
  resolve_timeout: 5m

route:
  receiver: 'slack'
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h

receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'YOUR_SLACK_WEBHOOK_URL'
        channel: '#alerts'
        send_resolved: true
EOF

# Docker Compose
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    restart: unless-stopped
  
  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    volumes:
      - ./alertmanager:/etc/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
    ports:
      - "9093:9093"
    restart: unless-stopped
  
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    command:
      - '--path.rootfs=/host'
    volumes:
      - '/:/host:ro,rslave'
    ports:
      - "9100:9100"
    restart: unless-stopped
  
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"
    restart: unless-stopped
  
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    restart: unless-stopped

volumes:
  prometheus-data:
  grafana-data:
EOF

# Start stack
docker-compose up -d

# Verify
echo "Prometheus: http://localhost:9090"
echo "Alertmanager: http://localhost:9093"
echo "Grafana: http://localhost:3000 (admin/admin)"
echo "Node Exporter: http://localhost:9100/metrics"
```

**Challenge:**
Enhance the stack:
1. Add more exporters
2. Configure recording rules
3. Set up Grafana dashboards
4. Implement custom alerts

---

### Exercise 2: Application Instrumentation (60 minutes)

**Objective:** Instrument application with Prometheus metrics

```python
# app.py - Flask application with Prometheus metrics
from flask import Flask, request
from prometheus_client import Counter, Histogram, Gauge, generate_latest
import time
import random

app = Flask(__name__)

# Define metrics
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_DURATION = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

ACTIVE_REQUESTS = Gauge(
    'http_requests_active',
    'Active HTTP requests'
)

ERROR_COUNT = Counter(
    'http_errors_total',
    'Total HTTP errors',
    ['method', 'endpoint', 'error_type']
)

# Middleware for metrics
@app.before_request
def before_request():
    request.start_time = time.time()
    ACTIVE_REQUESTS.inc()

@app.after_request
def after_request(response):
    request_duration = time.time() - request.start_time
    
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.path,
        status=response.status_code
    ).inc()
    
    REQUEST_DURATION.labels(
        method=request.method,
        endpoint=request.path
    ).observe(request_duration)
    
    ACTIVE_REQUESTS.dec()
    
    return response

# Routes
@app.route('/')
def index():
    return {'message': 'Hello, World!'}

@app.route('/api/users')
def users():
    # Simulate processing time
    time.sleep(random.uniform(0.1, 0.5))
    return {'users': ['Alice', 'Bob', 'Charlie']}

@app.route('/api/error')
def error():
    ERROR_COUNT.labels(
        method='GET',
        endpoint='/api/error',
        error_type='intentional'
    ).inc()
    return {'error': 'Something went wrong'}, 500

@app.route('/metrics')
def metrics():
    return generate_latest()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

```txt
# requirements.txt
flask==2.3.0
prometheus-client==0.17.0
```

```yaml
# Add to prometheus.yml
scrape_configs:
  - job_name: 'flask-app'
    static_configs:
      - targets: ['flask-app:5000']
```

**Challenge:**
Add advanced instrumentation:
1. Custom business metrics
2. Database query metrics
3. Cache hit/miss rates
4. Background job metrics

---

### Exercise 3: Advanced PromQL Queries (45 minutes)

**Objective:** Master complex PromQL queries

```promql
# 1. Request rate per second by endpoint
sum(rate(http_requests_total[5m])) by (endpoint)

# 2. 95th percentile response time
histogram_quantile(0.95, 
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le, endpoint)
)

# 3. Error rate percentage
sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m])) * 100

# 4. Top 5 endpoints by request count
topk(5, sum(rate(http_requests_total[5m])) by (endpoint))

# 5. Memory usage trend (next hour prediction)
predict_linear(node_memory_MemAvailable_bytes[1h], 3600)

# 6. CPU usage by core
100 - (avg by (cpu) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# 7. Network traffic (bytes per second)
rate(node_network_receive_bytes_total[5m])

# 8. Disk I/O operations per second
rate(node_disk_io_time_seconds_total[5m])

# 9. Container memory usage percentage
(container_memory_usage_bytes / container_spec_memory_limit_bytes) * 100

# 10. Service availability (uptime percentage)
avg_over_time(up[24h]) * 100

# 11. Request duration by quantile
histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) as p50,
histogram_quantile(0.90, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) as p90,
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) as p99

# 12. Apdex score (Application Performance Index)
(
  sum(rate(http_request_duration_seconds_bucket{le="0.5"}[5m]))
  +
  sum(rate(http_request_duration_seconds_bucket{le="2"}[5m])) / 2
)
/
sum(rate(http_request_duration_seconds_count[5m]))

# 13. Alert if disk will be full in 4 hours
predict_linear(node_filesystem_avail_bytes[1h], 4 * 3600) < 0

# 14. Requests per minute by status code
sum(increase(http_requests_total[1m])) by (status)

# 15. Memory leak detection (continuous growth)
deriv(process_resident_memory_bytes[1h]) > 0
```

**Challenge:**
Create custom queries for:
1. SLA compliance tracking
2. Cost optimization metrics
3. Capacity planning
4. Anomaly detection

---

## 🎯 Practice Projects

### Project 1: Complete Monitoring Solution
Build end-to-end monitoring:
- Multi-tier application monitoring
- Infrastructure metrics
- Custom business metrics
- Alerting and on-call rotation
- Grafana dashboards

### Project 2: Kubernetes Monitoring
Implement K8s monitoring:
- Cluster metrics
- Pod and container monitoring
- Service mesh observability
- Resource optimization
- Cost tracking

### Project 3: Multi-Cloud Monitoring
Monitor across clouds:
- AWS, Azure, GCP metrics
- Unified dashboards
- Cross-cloud alerting
- Cost analysis
- Performance comparison

### Project 4: SRE Dashboard
Build SRE platform:
- SLI/SLO tracking
- Error budgets
- Incident management
- Postmortem automation
- Capacity planning

---

## 📝 Best Practices

### 1. Metric Design
- Use consistent naming
- Choose appropriate metric types
- Add meaningful labels
- Avoid high cardinality
- Document metrics
- Version metric schemas

### 2. Query Optimization
- Use recording rules
- Limit time ranges
- Avoid regex when possible
- Use appropriate functions
- Cache frequent queries
- Monitor query performance

### 3. Storage Management
- Configure retention
- Use remote storage
- Implement downsampling
- Regular backups
- Monitor disk usage
- Plan capacity

### 4. Alerting Strategy
- Define clear SLOs
- Avoid alert fatigue
- Use proper severity levels
- Group related alerts
- Document runbooks
- Regular alert review

---

## ✅ Completion Checklist

### Week 1
- [ ] Understand Prometheus architecture
- [ ] Master metric types
- [ ] Learn PromQL basics
- [ ] Configure service discovery
- [ ] Set up exporters

### Week 2
- [ ] Configure alerting
- [ ] Implement HA setup
- [ ] Optimize queries
- [ ] Complete monitoring project
- [ ] Document best practices

---

**Next:** Continue with specialized topics or return to [Main README](../../README.md)