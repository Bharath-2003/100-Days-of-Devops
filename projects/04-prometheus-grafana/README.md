# Project 4: Set Up Full Monitoring Stack (Prometheus + Grafana)

## 🎯 Project Overview

**Difficulty:** Intermediate  
**Duration:** 6-8 hours  
**Technologies:** Prometheus, Grafana, Node Exporter, Alertmanager, Docker

### What You'll Build
A complete monitoring and observability stack with Prometheus for metrics collection, Grafana for visualization, Alertmanager for alerting, and various exporters for comprehensive system monitoring.

### Learning Objectives
- ✅ Set up Prometheus for metrics collection
- ✅ Configure Grafana dashboards
- ✅ Implement alerting with Alertmanager
- ✅ Use exporters for system metrics
- ✅ Create custom metrics
- ✅ Set up service discovery
- ✅ Implement high availability
- ✅ Configure retention and storage

### Prerequisites
- Docker and Docker Compose installed
- Basic understanding of metrics and monitoring
- Linux system or VM for testing
- Completed Projects 1-3 (optional but recommended)

---

## 📋 Project Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Monitoring Stack                              │
│                                                                   │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                    Grafana (Port 3000)                  │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │  Dashboards:                                      │  │    │
│  │  │  - System Overview                                │  │    │
│  │  │  - Application Metrics                            │  │    │
│  │  │  - Container Metrics                              │  │    │
│  │  │  - Alert Status                                   │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────┘    │
│                           │                                      │
│                           │ Query                                │
│                           ▼                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │              Prometheus (Port 9090)                     │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │  - Time Series Database                          │  │    │
│  │  │  - PromQL Query Engine                           │  │    │
│  │  │  - Service Discovery                             │  │    │
│  │  │  - Recording Rules                               │  │    │
│  │  │  - Alert Rules                                   │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────┘    │
│                           │                                      │
│                           │ Scrape Metrics                       │
│                           ▼                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                    Exporters                            │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │    │
│  │  │ Node Exporter│  │cAdvisor      │  │App Exporter  │ │    │
│  │  │ (Port 9100)  │  │(Port 8080)   │  │(Port 8000)   │ │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘ │    │
│  └────────────────────────────────────────────────────────┘    │
│                           │                                      │
│                           │ Send Alerts                          │
│                           ▼                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │              Alertmanager (Port 9093)                   │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │  - Alert Routing                                 │  │    │
│  │  │  - Grouping & Deduplication                      │  │    │
│  │  │  - Silencing                                     │  │    │
│  │  │  - Notification (Email, Slack, PagerDuty)       │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Step-by-Step Implementation

### Phase 1: Project Setup

#### Step 1.1: Create Project Structure

```bash
mkdir -p prometheus-grafana-stack
cd prometheus-grafana-stack

# Create directory structure
mkdir -p {prometheus,grafana,alertmanager}/{config,data}
mkdir -p exporters/node-exporter
mkdir -p dashboards
mkdir -p scripts

# Create files
touch docker-compose.yml
touch prometheus/config/prometheus.yml
touch prometheus/config/alerts.yml
touch alertmanager/config/alertmanager.yml
touch grafana/config/datasources.yml
touch grafana/config/dashboards.yml
```

#### Step 1.2: Create Docker Compose Configuration

**File: `docker-compose.yml`**
```yaml
version: '3.8'

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data:
  grafana_data:
  alertmanager_data:

services:
  # Prometheus
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/config:/etc/prometheus
      - prometheus_data:/prometheus
    networks:
      - monitoring
    depends_on:
      - alertmanager

  # Alertmanager
  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    restart: unless-stopped
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager/config:/etc/alertmanager
      - alertmanager_data:/alertmanager
    networks:
      - monitoring

  # Grafana
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_SERVER_ROOT_URL=http://localhost:3000
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/config/datasources.yml:/etc/grafana/provisioning/datasources/datasources.yml
      - ./grafana/config/dashboards.yml:/etc/grafana/provisioning/dashboards/dashboards.yml
      - ./dashboards:/var/lib/grafana/dashboards
    networks:
      - monitoring
    depends_on:
      - prometheus

  # Node Exporter
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    networks:
      - monitoring

  # cAdvisor (Container metrics)
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    restart: unless-stopped
    privileged: true
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    networks:
      - monitoring

  # Sample Application (for testing)
  sample-app:
    image: prom/prometheus:latest
    container_name: sample-app
    restart: unless-stopped
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - "9091:9090"
    networks:
      - monitoring
```

---

### Phase 2: Configure Prometheus

#### Step 2.1: Prometheus Configuration

**File: `prometheus/config/prometheus.yml`**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'monitoring-cluster'
    environment: 'production'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

# Load rules
rule_files:
  - 'alerts.yml'
  - 'recording_rules.yml'

# Scrape configurations
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: 'prometheus-server'

  # Node Exporter
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          instance: 'monitoring-host'

  # cAdvisor
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
        labels:
          instance: 'docker-host'

  # Alertmanager
  - job_name: 'alertmanager'
    static_configs:
      - targets: ['alertmanager:9093']

  # Sample Application
  - job_name: 'sample-app'
    static_configs:
      - targets: ['sample-app:9090']
        labels:
          app: 'sample-application'

  # Service Discovery (example for Kubernetes)
  # - job_name: 'kubernetes-pods'
  #   kubernetes_sd_configs:
  #     - role: pod
  #   relabel_configs:
  #     - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
  #       action: keep
  #       regex: true
```

#### Step 2.2: Alert Rules

**File: `prometheus/config/alerts.yml`**
```yaml
groups:
  - name: system_alerts
    interval: 30s
    rules:
      # High CPU Usage
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
          category: system
        annotations:
          summary: "High CPU usage detected on {{ $labels.instance }}"
          description: "CPU usage is above 80% (current value: {{ $value }}%)"

      # High Memory Usage
      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 5m
        labels:
          severity: warning
          category: system
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is above 85% (current value: {{ $value }}%)"

      # Disk Space Low
      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"}) * 100 < 15
        for: 10m
        labels:
          severity: critical
          category: storage
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Disk space is below 15% on {{ $labels.mountpoint }} (current: {{ $value }}%)"

      # Instance Down
      - alert: InstanceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
          category: availability
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "{{ $labels.job }} on {{ $labels.instance }} has been down for more than 2 minutes"

  - name: container_alerts
    interval: 30s
    rules:
      # Container High CPU
      - alert: ContainerHighCPU
        expr: rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100 > 80
        for: 5m
        labels:
          severity: warning
          category: container
        annotations:
          summary: "Container {{ $labels.name }} high CPU usage"
          description: "Container CPU usage is above 80% (current: {{ $value }}%)"

      # Container High Memory
      - alert: ContainerHighMemory
        expr: (container_memory_usage_bytes{name!=""} / container_spec_memory_limit_bytes{name!=""}) * 100 > 85
        for: 5m
        labels:
          severity: warning
          category: container
        annotations:
          summary: "Container {{ $labels.name }} high memory usage"
          description: "Container memory usage is above 85% (current: {{ $value }}%)"

      # Container Restart
      - alert: ContainerRestart
        expr: rate(container_last_seen{name!=""}[5m]) > 0
        for: 1m
        labels:
          severity: warning
          category: container
        annotations:
          summary: "Container {{ $labels.name }} restarted"
          description: "Container has restarted in the last 5 minutes"

  - name: application_alerts
    interval: 30s
    rules:
      # High Request Rate
      - alert: HighRequestRate
        expr: rate(http_requests_total[5m]) > 1000
        for: 5m
        labels:
          severity: warning
          category: application
        annotations:
          summary: "High request rate detected"
          description: "Request rate is above 1000 req/s (current: {{ $value }})"

      # High Error Rate
      - alert: HighErrorRate
        expr: (rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])) * 100 > 5
        for: 5m
        labels:
          severity: critical
          category: application
        annotations:
          summary: "High error rate detected"
          description: "Error rate is above 5% (current: {{ $value }}%)"

      # Slow Response Time
      - alert: SlowResponseTime
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
          category: performance
        annotations:
          summary: "Slow response time detected"
          description: "95th percentile response time is above 1s (current: {{ $value }}s)"
```

#### Step 2.3: Recording Rules

**File: `prometheus/config/recording_rules.yml`**
```yaml
groups:
  - name: recording_rules
    interval: 30s
    rules:
      # CPU Usage
      - record: instance:node_cpu_utilization:rate5m
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

      # Memory Usage
      - record: instance:node_memory_utilization:ratio
        expr: 1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)

      # Disk Usage
      - record: instance:node_disk_utilization:ratio
        expr: 1 - (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"})

      # Request Rate
      - record: job:http_requests:rate5m
        expr: sum by(job) (rate(http_requests_total[5m]))

      # Error Rate
      - record: job:http_errors:rate5m
        expr: sum by(job) (rate(http_requests_total{status=~"5.."}[5m]))
```

---

### Phase 3: Configure Alertmanager

**File: `alertmanager/config/alertmanager.yml`**
```yaml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@example.com'
  smtp_auth_username: 'your-email@gmail.com'
  smtp_auth_password: 'your-app-password'

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
    # Critical alerts
    - match:
        severity: critical
      receiver: 'critical-alerts'
      continue: true

    # Warning alerts
    - match:
        severity: warning
      receiver: 'warning-alerts'

    # System alerts
    - match:
        category: system
      receiver: 'system-alerts'

    # Application alerts
    - match:
        category: application
      receiver: 'app-alerts'

# Receivers
receivers:
  - name: 'default'
    email_configs:
      - to: 'team@example.com'
        headers:
          Subject: '[Monitoring] {{ .GroupLabels.alertname }}'

  - name: 'critical-alerts'
    email_configs:
      - to: 'oncall@example.com'
        headers:
          Subject: '[CRITICAL] {{ .GroupLabels.alertname }}'
    slack_configs:
      - api_url: 'YOUR_SLACK_WEBHOOK_URL'
        channel: '#alerts-critical'
        title: 'Critical Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'warning-alerts'
    email_configs:
      - to: 'team@example.com'
        headers:
          Subject: '[WARNING] {{ .GroupLabels.alertname }}'

  - name: 'system-alerts'
    email_configs:
      - to: 'sysadmin@example.com'

  - name: 'app-alerts'
    email_configs:
      - to: 'developers@example.com'

# Inhibition rules
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

---

### Phase 4: Configure Grafana

#### Step 4.1: Datasource Configuration

**File: `grafana/config/datasources.yml`**
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
    jsonData:
      timeInterval: "15s"
      queryTimeout: "60s"
```

#### Step 4.2: Dashboard Provisioning

**File: `grafana/config/dashboards.yml`**
```yaml
apiVersion: 1

providers:
  - name: 'Default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards
```

#### Step 4.3: Create System Dashboard

**File: `dashboards/system-overview.json`**
```json
{
  "dashboard": {
    "title": "System Overview",
    "panels": [
      {
        "title": "CPU Usage",
        "targets": [
          {
            "expr": "100 - (avg by(instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100"
          }
        ]
      },
      {
        "title": "Disk Usage",
        "targets": [
          {
            "expr": "(1 - (node_filesystem_avail_bytes{fstype!=\"tmpfs\"} / node_filesystem_size_bytes{fstype!=\"tmpfs\"})) * 100"
          }
        ]
      }
    ]
  }
}
```

---

## 🧪 Deployment and Testing

### Step 1: Start the Stack

```bash
# Start all services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f prometheus
docker-compose logs -f grafana
docker-compose logs -f alertmanager
```

### Step 2: Access Services

```bash
# Prometheus
open http://localhost:9090

# Grafana (admin/admin)
open http://localhost:3000

# Alertmanager
open http://localhost:9093

# Node Exporter metrics
curl http://localhost:9100/metrics
```

### Step 3: Verify Metrics Collection

```bash
# Check Prometheus targets
curl http://localhost:9090/api/v1/targets

# Query metrics
curl 'http://localhost:9090/api/v1/query?query=up'

# Check alert rules
curl http://localhost:9090/api/v1/rules
```

### Step 4: Test Alerting

```bash
# Generate high CPU load
stress --cpu 8 --timeout 300s

# Check alerts in Prometheus
open http://localhost:9090/alerts

# Check Alertmanager
open http://localhost:9093
```

---

## 📊 Key PromQL Queries

### System Metrics
```promql
# CPU Usage
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk Usage
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100

# Network Traffic
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])
```

### Container Metrics
```promql
# Container CPU
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100

# Container Memory
container_memory_usage_bytes{name!=""}

# Container Network
rate(container_network_receive_bytes_total[5m])
```

---

## 🎯 Success Criteria

- [ ] All services running
- [ ] Prometheus collecting metrics
- [ ] Grafana dashboards displaying data
- [ ] Alerts configured and firing
- [ ] Alertmanager routing notifications
- [ ] Exporters providing metrics
- [ ] Recording rules working
- [ ] Data retention configured

---

## 📚 Additional Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [PromQL Guide](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Alertmanager Documentation](https://prometheus.io/docs/alerting/latest/alertmanager/)

---

## 🎓 What You Learned

✅ Prometheus setup and configuration  
✅ Grafana dashboard creation  
✅ Alert rule configuration  
✅ Alertmanager setup  
✅ Metrics collection with exporters  
✅ PromQL query language  
✅ Service discovery  
✅ High availability setup  

---

## 🚀 Next Steps

1. **Project 5**: Build GitOps pipeline with ArgoCD
2. Add more exporters (MySQL, Redis, etc.)
3. Implement Thanos for long-term storage
4. Set up Loki for log aggregation
5. Add distributed tracing with Jaeger

---

**Project Status:** ✅ Complete  
**Last Updated:** 2024-01-16