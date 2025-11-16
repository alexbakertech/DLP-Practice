# Proxmox DLP Homelab Project

This repository contains all the artifacts for a complete, reproducible Data Loss Prevention (DLP) homelab environment deployed inside Proxmox. The lab demonstrates DLP in Motion, DLP at Rest, and DLP in Use across a fully isolated virtual environment.

## 2. Project Objectives

- Build an enterprise-modeled DLP testbed.
- Detect sensitive data exfiltration across network channels.
- Monitor unauthorized file modifications and USB activity.
- Produce correlated alerts through Suricata and Wazuh.
- Generate documentation, logs, and a final polished report.
- Publish reproducible artifacts in a GitHub repository.

## 3. System Architecture

The lab environment consists of:

- **dlp-gateway**: Suricata IPS for outbound inspection and DLP detection.
- **win11-client**: Endpoint workstation with seeded sensitive data.
- **wazuh-siem**: Wazuh all-in-one server for monitoring and alerting.
- **external-server**: Simulated public endpoint for exfiltration attempts.

## 4. Tooling Summary

- **Suricata**: Network-level DLP detection.
- **Wazuh**: Endpoint monitoring, FIM, USB detection.
- **Windows 11**: User workstation.
- **Proxmox**: Virtualization platform.
- **Ubuntu Server**: Gateway and external server roles.
- **(Optional) Microsoft Purview**: Enterprise-grade endpoint DLP features.

## 5. Build Plan

1.  **Deploy Wazuh SIEM**: Install Wazuh AIO on Ubuntu and configure FIM, USB monitoring, and optional active response.
2.  **Deploy Suricata Gateway**: Install Suricata, load DLP rule sets, and enable NAT routing.
3.  **Configure Windows Endpoint**: Seed files containing synthetic PII/PCI, install Wazuh agent, and configure monitoring.
4.  **External Server Setup**: Deploy Nginx, SSH, or FTP to act as an exfiltration target.

## 6. Test Scenarios

-   **Network Exfiltration**: Attempt HTTP uploads of sensitive files; Suricata detects the data patterns.
-   **File Integrity Violations**: Modify sensitive files; Wazuh FIM sends alerts.
-   **USB Exfiltration**: Copy files to USB; Wazuh detects insertion and monitors file writes.
-   **Optional: Microsoft Purview**: Block USB transfers and cloud uploads of sensitive data.

## 7. Deliverables

- Architecture diagram (PNG/SVG).
- Final report (3-5 pages).
- Evidence bundle including Suricata/Wazuh logs.
- GitHub repository with documentation, configs, and scripts.

## 8. GitHub Repository Structure

```
dlp-homelab/
├── README.md
├── docs/
├── configs/
├── scripts/
├── evidence/
└── reports/
```
