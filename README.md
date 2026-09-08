# Homelab-SOC-Wazuh-Deployment
# 🛡️ Enterprise SIEM & EDR Homelab: Threat Detection & Engineering

A self-hosted Security Operations Center (SOC) and detection engineering lab built on **Proxmox VE**, **Docker**, and **Wazuh SIEM/XDR**. This lab simulates real-world attack scenarios, monitors endpoint telemetry, and implements custom detection rules mapped to the **MITRE ATT&CK® framework**.

---

## 🏗️ Architecture Diagram


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

## 🛠️ Tools and Technologies Used

### 1. Virtualization & Infrastructure Layer
* **Proxmox Virtual Environment (PVE):**
  * **Type:** Enterprise Type-1 (Bare-Metal) Hypervisor.
  * **Role:** Hosts the virtualized ecosystem, provisions isolated Linux Containers (LXC), manages physical CPU/RAM allocation, and handles storage tiering (Fast SSD for OS/containers and 1 TB HDD for mass storage).
  * **Key Configurations:** Configured kernel namespaces (`nesting=1`, `keyctl=1`, `fuse=1`) and adjusted host-level virtual memory mapping parameters (`vm.max_map_count=262144`).

* **Debian GNU/Linux (Trixie/Bookworm):**
  * **Role:** Lightweight, low-overhead container operating system running the security telemetry agents and the container runtime engine.
  * **Key Configurations:** Multi-user permission model, Pluggable Authentication Modules (PAM), and static IPv4 networking.

---

### 2. Containerization & Orchestration Layer
* **Docker Engine:**
  * **Role:** Application container runtime powering the microservices architecture.
  * **Storage Driver:** Configured with `fuse-overlayfs` and overlay storage drivers to maintain isolated container filesystems within an unprivileged LXC environment.

* **Docker Compose:**
  * **Role:** Declarative multi-container orchestration.
  * **Usage:** Deployed and managed the single-node Wazuh stack (Indexer, Manager, and Dashboard) with automated health checks, internal container networking, and persistent volume mounting.

---

### 3. SIEM, EDR & Threat Intelligence Layer
* **Wazuh SIEM / XDR:**
  * **Wazuh Manager:** Central analysis engine responsible for receiving encrypted endpoint logs (TCP port 1514), decoding raw log messages, correlating events, and executing alerting rules.
  * **Wazuh Agent:** Endpoint daemon deployed on monitored targets performing real-time system inspection, log streaming, and security posture auditing.
  * **Custom Rule Engine:** Wrote and validated custom XML detection rules in `local_rules.xml` mapped to the **MITRE ATT&CK®** knowledge base.
  * **Testing Suite (`wazuh-logtest`):** Utilized built-in CLI log parser to test rule logic and verify decoding pipelines before pushing rules into production.

* **OpenSearch (Wazuh Indexer):**
  * **Role:** Highly scalable, distributed JSON-based search and document indexing engine that stores all ingested security telemetry and alert histories.

* **Wazuh Dashboard (OpenSearch Dashboards):**
  * **Role:** Web-based analyst interface providing real-time data visualization, security event triage, compliance tracking (PCI DSS, NIST 800-53, GDPR), and MITRE ATT&CK visualization.

---

### 4. Telemetry, Auditing & Host Security
* **File Integrity Monitoring (FIM / `syscheck`):**
  * **Role:** Automated cryptographic integrity auditing.
  * **Mechanism:** Monitors sensitive directories (`/etc`, `/usr/bin`) using real-time SHA256 checksum comparisons to detect unauthorized modifications, persistence scripts, or file defacements.

* **Linux Auth & PAM Subsystems (`/var/log/auth.log`):**
  * **Role:** Identity and access telemetry log source.
  * **Data Ingested:** Tracks user session lifecycle events, SSH authentication attempts, failed logins, and administrative privilege escalations (`sudo`).

* **Auditd / Sysmon for Linux:**
  * **Role:** High-fidelity kernel-level audit frameworks capturing process creation, execution arguments, command execution, and network connections.

---

### 5. Secure Networking & Zero Trust Access
* **Tailscale (WireGuard® Mesh VPN):**
  * **Role:** Encrypted out-of-band management access.
  * **Usage:** Facilitates remote access to the Proxmox Web GUI, container consoles, and Wazuh Dashboard from outside the local network without exposing administrative ports to the public internet.
