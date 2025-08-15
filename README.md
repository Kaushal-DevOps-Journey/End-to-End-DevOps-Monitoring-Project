# Ultimate-DevOps-Monitoring-Project

Monitoring Project with Prometheus, Alertmanager, Blackbox Exporter & Node Exporter

## 📌 Overview
This project sets up a monitoring system using **Prometheus** and related exporters to track the health of a Java application and server metrics.  
It uses **two virtual machines**:

- **VM1 (Monitoring Server)**:
  - Prometheus
  - Blackbox Exporter
  - Alertmanager

- **VM2 (Application Server)**:
  - Java-based application
  - Node Exporter

---

## 🖥️ Architecture
VM1: Monitoring Server
├── Prometheus
├── Blackbox Exporter
└── Alertmanager


VM2: Application Server
├── Java Application
└── Node Exporter

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
