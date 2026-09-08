# Homelab-SOC-Wazuh-Deployment
# 🛡️ Enterprise SIEM & EDR Homelab: Threat Detection & Engineering

A self-hosted Security Operations Center (SOC) and detection engineering lab built on **Proxmox VE**, **Docker**, and **Wazuh SIEM/XDR**. This lab simulates real-world attack scenarios, monitors endpoint telemetry, and implements custom detection rules mapped to the **MITRE ATT&CK® framework**.

---

## 🏗️ Architecture Diagram

```text
========================================================================================
                                    PHYSICAL HARDWARE
                      Dell Mini PC (Intel 8-Core, 16GB RAM, SSD + HDD)
========================================================================================
                                           │
                               ┌───────────▼───────────┐
                               │     Proxmox VE        │
                               │  (Hypervisor Layer)   │
                               └───────────┬───────────┘
                                           │
                      ┌────────────────────┴────────────────────┐
                      │ Bridge: vmbr0                           │ Tailscale Mesh
                      │ Local Subnet Routing                    │ (Remote Access)
                      └────────────────────┬────────────────────┘
                                           │
         ┌─────────────────────────────────▼─────────────────────────────────┐
         │                    CT1: Debian LXC Container                     │
         │                                                                   │
         │  ┌─────────────────────────────────────────────────────────────┐  │
         │  │                 Docker Compose Environment                  │  │
         │  │                                                             │  │
         │  │  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────┐ │  │
         │  │  │  Wazuh Indexer   │ │  Wazuh Manager   │ │    Wazuh     │ │  │
         │  │  │   (OpenSearch)   │◄┼─┤(Analysis Engine│─┼►│  Dashboard   │ │  │
         │  │  │  Stores & Indexes│ │  & Custom Rules) │ │   (Web GUI)  │ │  │
         │  │  └──────────────────┘ └────────▲─────────┘ └──────────────┘ │  │
         │  └────────────────────────────────┼────────────────────────────┘  │
         │                                   │ Port 1514 (Telemetry)        │
         │  ┌────────────────────────────────┴────────────────────────────┐  │
         │  │                   Wazuh Agent (CT1-Node)                    │  │
         │  │    • File Integrity Monitoring (FIM)                        │  │
         │  │    • Linux PAM / Auth Log Telemetry (/var/log/auth.log)     │  │
         │  │    • Security Configuration Assessment (SCA)                │  │
         │  └─────────────────────────────────────────────────────────────┘  │
         └───────────────────────────────────────────────────────────────────┘

