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
VM-1: Node Exporter + Java application --> Prometheus (VM-2) --> Alertmanager (VM-2) --> Gmail Alerts
                                       |
                                    Grafana (VM-2)
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
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets: ["<VM-1-IP>:9100"]
```

### 3️⃣ Setup Alertmanager on VM-2
**`alertmanager.yml` sample:**
```yaml
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'email-notifications'

receivers:
  - name: 'email-notifications'
    email_configs:
      - to: your_email@gmail.com
        from: sender_email@gmail.com
        smarthost: smtp.gmail.com:587
        auth_username: sender_email@gmail.com
        auth_identity: sender_email@gmail.com
        auth_password: "your_app_password"
        send_resolved: true

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
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

Login at: **http://<VM-2-IP>:3000** (Default user: `admin` / password: `admin`).

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
