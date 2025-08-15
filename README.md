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
│
▼
VM2: Application Server
├── Java Application
└── Node Exporter
