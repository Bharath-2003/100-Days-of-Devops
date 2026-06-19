# Grafana for DevOps

## 🎯 Overview

Grafana is an open-source analytics and monitoring platform that allows you to query, visualize, alert on, and understand your metrics. For DevOps engineers, mastering Grafana is essential for creating comprehensive monitoring dashboards and observability solutions.

**Duration:** 3 days (Part of Monitoring Week)
**Time Investment:** 3-4 hours/day
**Difficulty:** Beginner to Intermediate
**Prerequisites:** Basic understanding of metrics and monitoring concepts

---

## 📚 What You'll Learn

### Day 1: Grafana Fundamentals
- Grafana installation and setup
- Understanding the UI
- Data sources configuration
- Basic dashboard creation
- Panel types and visualizations

### Day 2: Advanced Dashboards
- Variables and templating
- Transformations
- Annotations
- Dashboard organization
- Sharing and exporting

### Day 3: Alerting and Integration
- Alert rules configuration
- Notification channels
- Integration with Prometheus
- Integration with Loki
- Best practices

---

## 🎓 Important Concepts

### 1. Grafana Architecture

```
Grafana Architecture
├─ Frontend (React)
│  ├─ Dashboard UI
│  ├─ Panel Editor
│  └─ Query Builder
│
├─ Backend (Go)
│  ├─ API Server
│  ├─ Alerting Engine
│  └─ Plugin System
│
├─ Data Sources
│  ├─ Prometheus
│  ├─ Loki
│  ├─ InfluxDB
│  ├─ Elasticsearch
│  ├─ MySQL/PostgreSQL
│  └─ CloudWatch, etc.
│
└─ Storage
   ├─ SQLite (default)
   ├─ MySQL
   └─ PostgreSQL
```

**Key Components:**
- **Dashboard:** Collection of panels
- **Panel:** Individual visualization
- **Data Source:** Where metrics come from
- **Query:** Request for data
- **Alert:** Notification based on conditions

---

### 2. Data Sources

```
Popular Data Sources
├─ Time Series Databases
│  ├─ Prometheus (metrics)
│  ├─ InfluxDB (metrics)
│  ├─ Graphite (metrics)
│  └─ TimescaleDB (metrics)
│
├─ Logging Systems
│  ├─ Loki (logs)
│  ├─ Elasticsearch (logs)
│  └─ CloudWatch Logs
│
├─ Tracing Systems
│  ├─ Jaeger (traces)
│  ├─ Tempo (traces)
│  └─ Zipkin (traces)
│
└─ Cloud Providers
   ├─ CloudWatch (AWS)
   ├─ Azure Monitor
   └─ Google Cloud Monitoring
```

**Data Source Configuration:**
```yaml
# datasources.yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
    
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    
  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3200
```

---

### 3. Panel Types and Visualizations

```
Panel Types
├─ Time Series
│  ├─ Graph (line, bar, points)
│  ├─ State Timeline
│  └─ Status History
│
├─ Stats
│  ├─ Stat (single value)
│  ├─ Gauge
│  └─ Bar Gauge
│
├─ Tables
│  ├─ Table
│  └─ Logs
│
├─ Misc
│  ├─ Heatmap
│  ├─ Pie Chart
│  ├─ Alert List
│  └─ Dashboard List
│
└─ Visualizations
   ├─ Text
   ├─ News
   └─ Plugin Panels
```

**Panel Configuration Example:**
```json
{
  "type": "timeseries",
  "title": "CPU Usage",
  "targets": [
    {
      "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
      "legendFormat": "{{instance}}"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "unit": "percent",
      "min": 0,
      "max": 100,
      "thresholds": {
        "steps": [
          {"value": 0, "color": "green"},
          {"value": 70, "color": "yellow"},
          {"value": 90, "color": "red"}
        ]
      }
    }
  }
}
```

---

### 4. Variables and Templating

```
Variable Types
├─ Query (from data source)
├─ Custom (manual list)
├─ Text box (user input)
├─ Constant (fixed value)
├─ Data source (select data source)
├─ Interval (time intervals)
├─ Ad hoc filters (dynamic filters)
└─ Global variables (built-in)
```

**Variable Examples:**
```
# Query variable
Name: instance
Query: label_values(up, instance)
Regex: .*
Multi-value: enabled
Include All: enabled

# Custom variable
Name: environment
Values: dev,staging,prod
Multi-value: disabled

# Interval variable
Name: interval
Values: 1m,5m,10m,30m,1h
Auto: enabled
```

**Using Variables in Queries:**
```promql
# Single variable
rate(http_requests_total{instance="$instance"}[5m])

# Multiple variables
rate(http_requests_total{instance=~"$instance", env="$environment"}[5m])

# With regex
rate(http_requests_total{instance=~"$instance.*"}[5m])

# All option
rate(http_requests_total{instance=~"$instance"}[5m])
```

---

### 5. Transformations

```
Transformation Types
├─ Filter
│  ├─ Filter by name
│  ├─ Filter by value
│  └─ Filter by query
│
├─ Organize
│  ├─ Organize fields
│  ├─ Rename by regex
│  └─ Group by
│
├─ Calculate
│  ├─ Add field from calculation
│  ├─ Binary operations
│  └─ Reduce
│
└─ Combine
   ├─ Merge
   ├─ Join by field
   └─ Concatenate
```

**Transformation Examples:**
```json
{
  "transformations": [
    {
      "id": "filterFieldsByName",
      "options": {
        "include": {
          "names": ["Time", "Value"]
        }
      }
    },
    {
      "id": "organize",
      "options": {
        "renameByName": {
          "Value": "CPU Usage"
        }
      }
    },
    {
      "id": "calculateField",
      "options": {
        "mode": "binary",
        "reduce": {
          "reducer": "mean"
        }
      }
    }
  ]
}
```

---

### 6. Alerting

```
Alerting Architecture
├─ Alert Rules
│  ├─ Query
│  ├─ Conditions
│  ├─ Evaluation interval
│  └─ For duration
│
├─ Contact Points
│  ├─ Email
│  ├─ Slack
│  ├─ PagerDuty
│  ├─ Webhook
│  └─ Custom integrations
│
├─ Notification Policies
│  ├─ Routing rules
│  ├─ Grouping
│  ├─ Timing
│  └─ Mute timings
│
└─ Silences
   └─ Temporary muting
```

**Alert Rule Example:**
```yaml
# alert-rules.yaml
apiVersion: 1

groups:
  - name: system_alerts
    interval: 1m
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"
      
      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 90
        for: 5m
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}% on {{ $labels.instance }}"
```

---

### 7. Dashboard Best Practices

```
Dashboard Organization
├─ Folder Structure
│  ├─ By Team (Platform, Backend, Frontend)
│  ├─ By Service (API, Database, Cache)
│  └─ By Environment (Dev, Staging, Prod)
│
├─ Dashboard Layout
│  ├─ Overview at top
│  ├─ Key metrics prominent
│  ├─ Details below
│  └─ Logs at bottom
│
├─ Naming Conventions
│  ├─ [Team] Service - Metric Type
│  ├─ Example: [Platform] Kubernetes - Cluster Overview
│  └─ Use consistent prefixes
│
└─ Performance
   ├─ Limit queries per panel
   ├─ Use appropriate time ranges
   ├─ Optimize query intervals
   └─ Use caching when possible
```

---

## 🔨 Hands-On Exercises

### Exercise 1: Install and Configure Grafana (30 minutes)

**Objective:** Set up Grafana with Docker

```bash
# 1. Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      - ./provisioning:/etc/grafana/provisioning
    networks:
      - monitoring

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    networks:
      - monitoring

volumes:
  grafana-data:
  prometheus-data:

networks:
  monitoring:
    driver: bridge
EOF

# 2. Create Prometheus config
mkdir -p provisioning/datasources
cat > prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
EOF

# 3. Provision Prometheus data source
cat > provisioning/datasources/prometheus.yaml << 'EOF'
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
EOF

# 4. Start services
docker-compose up -d

# 5. Wait for services to start
sleep 10

# 6. Check Grafana
curl http://localhost:3000/api/health

# 7. Access Grafana
echo "Grafana: http://localhost:3000"
echo "Username: admin"
echo "Password: admin123"

# 8. Check Prometheus
curl http://localhost:9090/api/v1/targets
```

**Challenge:**
Set up Grafana with:
1. Multiple data sources
2. Custom configuration
3. LDAP authentication
4. HTTPS enabled

---

### Exercise 2: Create System Monitoring Dashboard (60 minutes)

**Objective:** Build comprehensive system monitoring dashboard

```bash
# 1. Create dashboard JSON
cat > system-dashboard.json << 'EOF'
{
  "dashboard": {
    "title": "System Monitoring",
    "tags": ["system", "monitoring"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "CPU Usage",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0},
        "targets": [
          {
            "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "{{instance}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "min": 0,
            "max": 100
          }
        }
      },
      {
        "id": 2,
        "title": "Memory Usage",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0},
        "targets": [
          {
            "expr": "(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100",
            "legendFormat": "{{instance}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "min": 0,
            "max": 100
          }
        }
      },
      {
        "id": 3,
        "title": "Disk Usage",
        "type": "gauge",
        "gridPos": {"h": 8, "w": 8, "x": 0, "y": 8},
        "targets": [
          {
            "expr": "(node_filesystem_size_bytes{mountpoint=\"/\"} - node_filesystem_free_bytes{mountpoint=\"/\"}) / node_filesystem_size_bytes{mountpoint=\"/\"} * 100"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "steps": [
                {"value": 0, "color": "green"},
                {"value": 70, "color": "yellow"},
                {"value": 90, "color": "red"}
              ]
            }
          }
        }
      },
      {
        "id": 4,
        "title": "Network Traffic",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 16, "x": 8, "y": 8},
        "targets": [
          {
            "expr": "rate(node_network_receive_bytes_total[5m])",
            "legendFormat": "{{device}} - RX"
          },
          {
            "expr": "rate(node_network_transmit_bytes_total[5m])",
            "legendFormat": "{{device}} - TX"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "Bps"
          }
        }
      }
    ]
  }
}
EOF

# 2. Import dashboard via API
curl -X POST http://admin:admin123@localhost:3000/api/dashboards/db \
  -H "Content-Type: application/json" \
  -d @system-dashboard.json

# 3. Create dashboard with variables
cat > advanced-dashboard.json << 'EOF'
{
  "dashboard": {
    "title": "Advanced System Monitoring",
    "templating": {
      "list": [
        {
          "name": "instance",
          "type": "query",
          "datasource": "Prometheus",
          "query": "label_values(up, instance)",
          "multi": true,
          "includeAll": true
        },
        {
          "name": "interval",
          "type": "interval",
          "query": "1m,5m,10m,30m,1h",
          "auto": true
        }
      ]
    },
    "panels": [
      {
        "title": "CPU Usage - $instance",
        "targets": [
          {
            "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\",instance=~\"$instance\"}[$interval])) * 100)"
          }
        ]
      }
    ]
  }
}
EOF

# 4. Import advanced dashboard
curl -X POST http://admin:admin123@localhost:3000/api/dashboards/db \
  -H "Content-Type: application/json" \
  -d @advanced-dashboard.json
```

**Challenge:**
Create dashboards for:
1. Application metrics
2. Database performance
3. Container metrics
4. Custom business metrics

---

### Exercise 3: Configure Alerting (45 minutes)

**Objective:** Set up alert rules and notifications

```bash
# 1. Create alert rules
mkdir -p provisioning/alerting
cat > provisioning/alerting/rules.yaml << 'EOF'
apiVersion: 1

groups:
  - name: system_alerts
    interval: 1m
    rules:
      - uid: high_cpu
        title: High CPU Usage
        condition: A
        data:
          - refId: A
            queryType: ''
            relativeTimeRange:
              from: 300
              to: 0
            datasourceUid: prometheus
            model:
              expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
              refId: A
        noDataState: NoData
        execErrState: Error
        for: 5m
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"
        labels:
          severity: warning
      
      - uid: high_memory
        title: High Memory Usage
        condition: A
        data:
          - refId: A
            queryType: ''
            relativeTimeRange:
              from: 300
              to: 0
            datasourceUid: prometheus
            model:
              expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
              refId: A
        noDataState: NoData
        execErrState: Error
        for: 5m
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}%"
        labels:
          severity: critical
EOF

# 2. Configure contact points
cat > provisioning/alerting/contact-points.yaml << 'EOF'
apiVersion: 1

contactPoints:
  - orgId: 1
    name: email
    receivers:
      - uid: email1
        type: email
        settings:
          addresses: admin@example.com
  
  - orgId: 1
    name: slack
    receivers:
      - uid: slack1
        type: slack
        settings:
          url: https://hooks.slack.com/services/YOUR/WEBHOOK/URL
          text: |
            {{ range .Alerts }}
            *Alert:* {{ .Labels.alertname }}
            *Summary:* {{ .Annotations.summary }}
            *Description:* {{ .Annotations.description }}
            {{ end }}
EOF

# 3. Configure notification policies
cat > provisioning/alerting/policies.yaml << 'EOF'
apiVersion: 1

policies:
  - orgId: 1
    receiver: email
    group_by: ['alertname', 'instance']
    group_wait: 30s
    group_interval: 5m
    repeat_interval: 4h
    routes:
      - receiver: slack
        matchers:
          - severity = critical
        group_wait: 10s
        repeat_interval: 1h
EOF

# 4. Restart Grafana to apply changes
docker-compose restart grafana

# 5. Test alert
# Trigger high CPU
stress --cpu 4 --timeout 60s

# 6. Check alert status via API
curl http://admin:admin123@localhost:3000/api/alertmanager/grafana/api/v2/alerts
```

**Challenge:**
Configure advanced alerting:
1. Multiple notification channels
2. Alert routing based on labels
3. Silences and mute timings
4. Custom webhook integrations

---

### Exercise 4: Create Application Dashboard (60 minutes)

**Objective:** Build dashboard for application monitoring

```bash
# 1. Create application metrics dashboard
cat > app-dashboard.json << 'EOF'
{
  "dashboard": {
    "title": "Application Monitoring",
    "tags": ["application", "api"],
    "templating": {
      "list": [
        {
          "name": "service",
          "type": "query",
          "datasource": "Prometheus",
          "query": "label_values(http_requests_total, service)",
          "multi": false
        },
        {
          "name": "status",
          "type": "custom",
          "query": "200,400,500",
          "multi": true,
          "includeAll": true
        }
      ]
    },
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{service=\"$service\"}[5m])) by (method, endpoint)",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      },
      {
        "id": 2,
        "title": "Error Rate",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{service=\"$service\",status=~\"5..\"}[5m])) / sum(rate(http_requests_total{service=\"$service\"}[5m])) * 100",
            "legendFormat": "Error Rate"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent"
          }
        }
      },
      {
        "id": 3,
        "title": "Response Time (p95)",
        "type": "timeseries",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{service=\"$service\"}[5m])) by (le, endpoint))",
            "legendFormat": "{{endpoint}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "s"
          }
        }
      },
      {
        "id": 4,
        "title": "Status Code Distribution",
        "type": "piechart",
        "targets": [
          {
            "expr": "sum by (status) (rate(http_requests_total{service=\"$service\",status=~\"$status\"}[5m]))",
            "legendFormat": "{{status}}"
          }
        ]
      },
      {
        "id": 5,
        "title": "Top Endpoints by Traffic",
        "type": "table",
        "targets": [
          {
            "expr": "topk(10, sum by (endpoint) (rate(http_requests_total{service=\"$service\"}[5m])))",
            "format": "table",
            "instant": true
          }
        ],
        "transformations": [
          {
            "id": "organize",
            "options": {
              "renameByName": {
                "Value": "Requests/sec"
              }
            }
          }
        ]
      }
    ]
  }
}
EOF

# 2. Import dashboard
curl -X POST http://admin:admin123@localhost:3000/api/dashboards/db \
  -H "Content-Type: application/json" \
  -d @app-dashboard.json

# 3. Create RED metrics dashboard (Rate, Errors, Duration)
cat > red-dashboard.json << 'EOF'
{
  "dashboard": {
    "title": "RED Metrics - $service",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [{
          "expr": "sum(rate(http_requests_total{service=\"$service\"}[5m]))"
        }]
      },
      {
        "title": "Error Rate",
        "targets": [{
          "expr": "sum(rate(http_requests_total{service=\"$service\",status=~\"5..\"}[5m])) / sum(rate(http_requests_total{service=\"$service\"}[5m]))"
        }]
      },
      {
        "title": "Duration (p50, p95, p99)",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket{service=\"$service\"}[5m])) by (le))",
            "legendFormat": "p50"
          },
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{service=\"$service\"}[5m])) by (le))",
            "legendFormat": "p95"
          },
          {
            "expr": "histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket{service=\"$service\"}[5m])) by (le))",
            "legendFormat": "p99"
          }
        ]
      }
    ]
  }
}
EOF

curl -X POST http://admin:admin123@localhost:3000/api/dashboards/db \
  -H "Content-Type: application/json" \
  -d @red-dashboard.json
```

**Challenge:**
Create specialized dashboards:
1. Database performance
2. Cache hit rates
3. Queue metrics
4. Business KPIs

---

### Exercise 5: Dashboard as Code (45 minutes)

**Objective:** Manage dashboards with version control

```bash
# 1. Create dashboard provisioning directory
mkdir -p provisioning/dashboards

# 2. Create dashboard provider config
cat > provisioning/dashboards/dashboards.yaml << 'EOF'
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards
      foldersFromFilesStructure: true
EOF

# 3. Create Kubernetes dashboard
cat > provisioning/dashboards/kubernetes.json << 'EOF'
{
  "dashboard": {
    "title": "Kubernetes Cluster Overview",
    "tags": ["kubernetes", "cluster"],
    "panels": [
      {
        "title": "Cluster CPU Usage",
        "targets": [{
          "expr": "sum(rate(container_cpu_usage_seconds_total[5m])) / sum(machine_cpu_cores) * 100"
        }]
      },
      {
        "title": "Cluster Memory Usage",
        "targets": [{
          "expr": "sum(container_memory_working_set_bytes) / sum(machine_memory_bytes) * 100"
        }]
      },
      {
        "title": "Pod Count",
        "targets": [{
          "expr": "count(kube_pod_info)"
        }]
      },
      {
        "title": "Node Status",
        "targets": [{
          "expr": "kube_node_status_condition{condition=\"Ready\",status=\"true\"}"
        }]
      }
    ]
  }
}
EOF

# 4. Create backup script
cat > backup-dashboards.sh << 'EOF'
#!/bin/bash

GRAFANA_URL="http://localhost:3000"
GRAFANA_USER="admin"
GRAFANA_PASS="admin123"
BACKUP_DIR="./dashboard-backups"

mkdir -p $BACKUP_DIR

# Get all dashboards
dashboards=$(curl -s -u $GRAFANA_USER:$GRAFANA_PASS \
  $GRAFANA_URL/api/search?type=dash-db | \
  jq -r '.[] | .uid')

# Backup each dashboard
for uid in $dashboards; do
  echo "Backing up dashboard: $uid"
  curl -s -u $GRAFANA_USER:$GRAFANA_PASS \
    $GRAFANA_URL/api/dashboards/uid/$uid | \
    jq '.dashboard' > $BACKUP_DIR/$uid.json
done

echo "Backup complete!"
EOF

chmod +x backup-dashboards.sh

# 5. Create restore script
cat > restore-dashboards.sh << 'EOF'
#!/bin/bash

GRAFANA_URL="http://localhost:3000"
GRAFANA_USER="admin"
GRAFANA_PASS="admin123"
BACKUP_DIR="./dashboard-backups"

for file in $BACKUP_DIR/*.json; do
  echo "Restoring: $file"
  dashboard=$(cat $file)
  payload=$(jq -n --argjson dashboard "$dashboard" '{dashboard: $dashboard, overwrite: true}')
  
  curl -X POST -u $GRAFANA_USER:$GRAFANA_PASS \
    -H "Content-Type: application/json" \
    -d "$payload" \
    $GRAFANA_URL/api/dashboards/db
done

echo "Restore complete!"
EOF

chmod +x restore-dashboards.sh

# 6. Run backup
./backup-dashboards.sh

# 7. Initialize git repository
git init
git add provisioning/ *.sh
git commit -m "Initial dashboard configuration"
```

**Challenge:**
Implement dashboard management:
1. Automated backups
2. Version control
3. CI/CD for dashboards
4. Dashboard testing

---

## 🎯 Practice Projects

### Project 1: Complete Observability Stack
Build full monitoring solution:
- Grafana for visualization
- Prometheus for metrics
- Loki for logs
- Tempo for traces
- Alert manager for notifications
- Service mesh integration

### Project 2: Multi-Tenant Grafana
Set up multi-tenant environment:
- Organization separation
- Team-specific dashboards
- Role-based access control
- Custom branding per org
- Isolated data sources

### Project 3: Custom Plugin Development
Create custom Grafana plugin:
- Data source plugin
- Panel plugin
- App plugin
- Custom visualizations
- Backend integration

### Project 4: Grafana at Scale
Deploy enterprise Grafana:
- High availability setup
- Load balancing
- Database clustering
- Caching strategy
- Performance optimization

---

## 📝 Best Practices

### 1. Dashboard Design
- Keep dashboards focused and simple
- Use consistent color schemes
- Group related panels
- Add descriptions and documentation
- Use appropriate time ranges
- Optimize query performance

### 2. Query Optimization
- Use recording rules for complex queries
- Limit time ranges appropriately
- Use variables for flexibility
- Cache query results
- Avoid expensive aggregations
- Use appropriate intervals

### 3. Alerting
- Set meaningful thresholds
- Avoid alert fatigue
- Use appropriate evaluation intervals
- Group related alerts
- Document alert responses
- Test alerts regularly

### 4. Organization
- Use folders for organization
- Tag dashboards appropriately
- Version control dashboards
- Document dashboard purpose
- Share dashboards with teams
- Regular dashboard reviews

---

## ✅ Completion Checklist

### Day 1
- [ ] Install Grafana
- [ ] Configure data sources
- [ ] Create first dashboard
- [ ] Add basic panels
- [ ] Understand panel types

### Day 2
- [ ] Use variables
- [ ] Apply transformations
- [ ] Add annotations
- [ ] Organize dashboards
- [ ] Share dashboards

### Day 3
- [ ] Configure alerts
- [ ] Set up notifications
- [ ] Integrate with Prometheus
- [ ] Create alert rules
- [ ] Test alerting

---

**Next:** [Prometheus →](../prometheus/README.md)