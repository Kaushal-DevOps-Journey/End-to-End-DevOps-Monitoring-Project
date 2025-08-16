# Monitoring & Alerting System with Prometheus, Grafana, Blackbox Exporter & Node Exporter and Alertmanager

## 📌 Project Overview
This project sets up a monitoring and alerting system for an **Java application** and **Node Exporter** running on **VM-1** using **Prometheus**, **Blackbox Exporter**, **Alertmanager**, and **Grafana** hosted on **VM-2**. Alerts are sent via Gmail when specified conditions are met.
 

It uses **two virtual machines**:

- **VM1 (Monitoring Server)**:
  - Prometheus
  - Blackbox Exporter
  - Alertmanager
  - Grafana

- **VM2 (Application Server)**:
  - Java-based application
  - Node Exporter

---

<img width="1771" height="991" alt="image" src="https://github.com/user-attachments/assets/d49769a6-131e-4af5-a504-0b9a7bd3fa17" />


## 🖥 Architecture
- **VM-1**:  
  - **Node Exporter** → Collects system metrics (CPU, memory, disk usage).  
  - **Java application** → Application being monitored.

- **VM-2**:  
  - **Prometheus** → Scrapes metrics from Node Exporter and Blackbox Exporter.
  - **Blackbox Exporter** → Collects application metrics
  - **Alertmanager** → Sends alerts via Gmail when conditions are triggered.  
  - **Grafana** → Visualizes metrics with custom dashboards.

- **Email (Gmail)** → Receives alert notifications.

---

## alert_rules.yml  -- Criteria for sending alerts to mail address
```bash
groups:
- name: alert_rules                   # Name of the alert rules group
  rules:

  - alert: InstanceDown
    expr: up == 0                     # Expression to detect instance down
    for: 1m
    labels:
      severity: "critical"
    annotations:
      summary: "Endpoint {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."

  - alert: WebsiteDown
    expr: probe_success == 0          # Expression to detect website down
    for: 1m
    labels:
      severity: "critical"
    annotations:
      summary: "Website down"
      description: "The website at {{ $labels.instance }} is down."

  - alert: HostOutOfMemory
    expr: (node_memory_MemAvailable / node_memory_MemTotal) * 100 < 25  # Detect low memory
    for: 5m
    labels:
      severity: "warning"
    annotations:
      summary: "Host out of memory (instance {{ $labels.instance }})"
      description: "Node memory is filling up (< 25% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

  - alert: HostOutOfDiskSpace
    expr: (node_filesystem_avail{mountpoint="/"} * 100) / node_filesystem_size{mountpoint="/"} < 50  # Low disk space
    for: 1s
    labels:
      severity: "warning"
    annotations:
      summary: "Host out of disk space (instance {{ $labels.instance }})"
      description: "Disk is almost full (< 50% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

  - alert: HostHighCpuLoad
    expr: (1 - avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100 > 80  # High CPU load
    for: 5m
    labels:
      severity: "warning"
    annotations:
      summary: "Host high CPU load (instance {{ $labels.instance }})"
      description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

  - alert: ServiceUnavailable
    expr: up{job="node_exporter"} == 0  # Service unavailability
    for: 2m
    labels:
      severity: "critical"
    annotations:
      summary: "Service Unavailable (instance {{ $labels.instance }})"
      description: "The service {{ $labels.job }} is not available\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

  - alert: HighMemoryUsage
    expr: (node_memory_Active / node_memory_MemTotal) * 100 > 90  # High memory usage
    for: 10m
    labels:
      severity: "critical"
    annotations:
      summary: "High Memory Usage (instance {{ $labels.instance }})"
      description: "Memory usage is > 90%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

  - alert: FileSystemFull
    expr: (node_filesystem_avail / node_filesystem_size) * 100 < 10  # File system almost full
    for: 5m
    labels:
      severity: "critical"
    annotations:
      summary: "File System Almost Full (instance {{ $labels.instance }})"
      description: "File system has < 10% free space\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



---

## 📊 Architecture Diagram
```
VM-1: 
  • Node Exporter --> Prometheus (VM-2)
  • Java Application --> Blackbox Exporter (VM-2) --> Prometheus (VM-2)

VM-2: 
  • Prometheus --> Alertmanager --> Gmail Alerts
             |
             --> Grafana (dashboards)

```

---

## ⚙ Installation Steps

### 1️⃣ Setup Node Exporter on VM-1
```bash
# Download Node Exporter
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvf node_exporter-*.tar.gz
cd node_exporter-*

# Start Node Exporter
./node_exporter &
```

### 2️⃣ Setup Prometheus on VM-2
```bash
# Download Prometheus
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-2.47.0.linux-amd64.tar.gz
tar xvf prometheus-*.tar.gz
cd prometheus-*

# Start Prometheus
./prometheus &
```

**`prometheus.yml` sample:**
```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - 18.214.100.108:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "alert_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
      - targets: ["localhost:9090"]
        # The label name is added as a label `label_name=<label_value>` to any timeseries scraped from this config.
        labels:
          app: "prometheus"

  - job_name: "node_exporter" # Job name for node exporter
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'
    static_configs:
      - targets: ["54.196.207.20:9100"] # Target node exporter endpoint

  - job_name: "blackbox" # Job name for blackbox exporter
    metrics_path: /probe # Path for blackbox probe
    params:
      module: [http_2xx] # Module to look for HTTP 200 response
    static_configs:
      - targets:
          - "http://prometheus.io" # HTTP target
          - "https://prometheus.io" # HTTPS target
          - "http://54.196.207.20:8080" # HTTP target with port 8080
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 18.214.100.108:9115 # Blackbox exporter address

```

### 3️⃣ Setup Alertmanager on VM-2
**`alertmanager.yml`:**
```yaml
route:
  group_by:
    - alertname
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: email-notifications
receivers:
  - name: email-notifications
    email_configs:
      - to: kaushalkumar5407@gmail.com
        from: Monitoring_Test@gmail.com
        smarthost: smtp.gmail.com:587
        auth_username: kaushalkumar5407@gmail.com
        auth_identity: kaushalkumar5407@gmail.com
        auth_password: hbgj pagz lnwy ayph
        send_resolved: true


inhibit_rules:
  - source_match:
      severity: 'critical'            # Source alert severity
    target_match:
      severity: 'warning'             # Target alert severity
    equal: ['alertname', 'dev', 'instance']  # Fields to match


```

Start Alertmanager:
```bash
./alertmanager &
```

### 4️⃣ Setup Grafana on VM-2
```bash
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_10.0.3_amd64.deb
sudo dpkg -i grafana_10.0.3_amd64.deb

# Start Grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Login at: 
```
**http://<VM-2-IP>:3000** (Default user: `admin` / password: `admin`).
```
### 5 Setup Blackbox Exporter on VM-2
```bash
# Download Prometheus
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar xvf blackbox_exporter-*.tar.gz
cd blackbox_exporter-*

# Start Prometheus
./blackbox_exporter &
```
---

## 🚨 Alerts
- Email notifications sent via Gmail when system metrics exceed thresholds.
- Example alert: CPU usage > 80% for 5 minutes.

---

## 📧 Gmail Setup
1. Enable **2-Step Verification** in your Gmail account.
2. Generate an **App Password** under Security settings.
3. Use that password in `alertmanager.yml`.

---

## 📈 Grafana Dashboards
- Import Prometheus as a data source.
- Create panels to visualize CPU, memory, and disk usage.



---

## ✅ Summary
- **VM-1** runs Java application & Node Exporter.
- **VM-2** runs Prometheus, Blackbox Exporter, Alertmanager, and Grafana.
- Email alerts sent via Gmail when alerts trigger.
- Grafana visualizes metrics from Prometheus.

---

## 🔗 Useful Links
- [Prometheus Docs](https://prometheus.io/docs/)
- [Alertmanager Docs](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [Grafana Docs](https://grafana.com/docs/)
