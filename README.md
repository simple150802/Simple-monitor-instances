# System Monitoring with Prometheus, Grafana, and Node Exporter for 1 host
## Step 1: Install docker 
```bash
sudo apt-get update -y
sudo apt-get upgrade -y


sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common


curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -


sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) stable"

sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

sudo usermod -aG docker $USER
```

## Step 2: Create directory monitor and write docker-compose
```bash
mkdir -p ~/monitoring
cd ~/monitoring
```
- Write docker compose 
```bash
version: "3.7"

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    restart: always

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus
    restart: always

  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    ports:
      - "9100:9100"
    restart: always
```
- Trong cùng thư mục ~/monitoring tạo prometheus.yml
```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["node_exporter:9100"]
```
## Step 3: Start Prometheus Container
```bash
docker compose up -d
```
- Check status of container
## Step 4: Verify Integration
- Access Prometheus Web UI: Open your browser and go to http://localhost:9090. (if using public IP using it instead of localhost)
- Check Targets: Navigate to the “Targets” page in the Prometheus UI. You should see both ‘prometheus’ and ‘node-exporter’ listed as targets, indicating successful integration.
- Grafana: http://localhost:3000 (or IP public if using them) (user/pass: admin/admin)
## Step 5: Visualize data
```bash
Choose connection -> data sources -> choose Prometheus -> enter Prometheus server URL and Save.
Go to Grafana → Dashboards → Import 
```
- Paste this With uid is the name of data source 
```bash
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "prometheus",
          "uid": "prometheus"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "gnetId": null,
  "graphTooltip": 0,
  "id": null,
  "links": [],
  "panels": [
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 0
      },
      "id": 1,
      "title": "CPU Usage (%)",
      "type": "timeseries",
      "targets": [
        {
          "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
          "legendFormat": "CPU Usage",
          "refId": "A"
        }
      ]
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent"
        }
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 0
      },
      "id": 2,
      "title": "Memory Usage (%)",
      "type": "timeseries",
      "targets": [
        {
          "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100",
          "legendFormat": "Memory Usage",
          "refId": "A"
        }
      ]
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent"
        }
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 8
      },
      "id": 3,
      "title": "Disk Usage (%)",
      "type": "timeseries",
      "targets": [
        {
          "expr": "(1 - (node_filesystem_avail_bytes{fstype!=\"tmpfs\",fstype!=\"overlay\"} / node_filesystem_size_bytes{fstype!=\"tmpfs\",fstype!=\"overlay\"})) * 100",
          "legendFormat": "{{mountpoint}}",
          "refId": "A"
        }
      ]
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "short"
        }
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 8
      },
      "id": 4,
      "title": "Load Average (1m)",
      "type": "timeseries",
      "targets": [
        {
          "expr": "node_load1",
          "legendFormat": "Load 1m",
          "refId": "A"
        }
      ]
    }
  ],
  "refresh": "10s",
  "schemaVersion": 30,
  "style": "dark",
  "tags": ["node_exporter", "system", "vm"],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-15m",
    "to": "now"
  },
  "timezone": "",
  "title": "VM System Metrics",
  "uid": "vm-system-metrics",
  "version": 1
}
```
- If  want to download metric with CSV style, In the upper right corner of the panel → click the panel title choose Inspect → Data and dowload csv form
