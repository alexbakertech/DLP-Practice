# Proxmox DLP Homelab Project

This repository contains all the artifacts for a complete, reproducible Data Loss Prevention (DLP) homelab environment deployed inside Proxmox. The lab models a segmented enterprise network using pfSense, VLANs, Suricata, Wazuh, and a Windows client. It demonstrates DLP in Motion, DLP at Rest, and DLP in Use across a fully isolated virtual environment.

## 1. Index

- Visit the project's [Documentation Homepage](docs/INDEX.md)

## 2. Project Objectives

- Build an enterprise-modeled, VLAN-segmented DLP testbed.
- Detect sensitive data exfiltration across network channels.
- Monitor unauthorized file modifications, USB activity, and endpoint behavior.
- Produce correlated alerts using Suricata (network) and Wazuh (endpoint/SIEM).
- Generate documentation, logs, and a polished final report.
- Publish reproducible artifacts in a structured GitHub repository.

## 3. System Architecture

The lab is deployed as a multi-zone virtual network enforced by pfSense and consists of:

### Core Components

- **pfSense Router/Firewall**  
  Provides network segmentation, VLAN routing, and controlled east–west and egress paths.

- **dlp-gateway** (Ubuntu + Suricata)  
  Inline inspection gateway for DLP in Motion. Handles outbound inspection, pattern matching, and alert generation.

- **win10-client**  
  Corporate workstation used for endpoint-based DLP scenarios. Contains seeded synthetic PII/PCI content.

- **wazuh-siem**  
  Wazuh all-in-one deployment providing File Integrity Monitoring, USB monitoring, log aggregation, and alert correlation.

- **external-server**  
  Simulated “public” target reachable only through the pfSense-controlled DMZ VLAN. Used to test exfiltration scenarios.

### Network Zones (Proxmox VLANs)

- **VLAN 810** – Client Network (`win11-client`, `wazuh-siem`)
- **VLAN 820** – Security Gateway (`dlp-gateway`)
- **VLAN 830** – DMZ (`external-server`)
- **VLAN 899** – Management (optional / future extension)

All traffic from the client network destined for external zones must traverse:

```
win11-client → pfSense → dlp-gateway → pfSense → DMZ/external-server
```


This design mirrors corporate segmentation and supports realistic DLP inspection points.

## 4. Tooling Summary

- **pfSense**: VLAN segmentation, NAT, firewall policy, and traffic control.
- **Suricata**: Network-level DLP detection using DLP and exfiltration rule sets.
- **Wazuh**: Endpoint monitoring, File Integrity Monitoring (FIM), USB activity detection, and optional active response.
- **Windows 10**: Endpoint workstation for testing user-driven exfiltration attempts.
- **Proxmox VE**: Virtualization and distributed networking fabric for the entire project.
- **Ubuntu Server**: Base OS for Suricata gateway and external server.
- **(Optional) Microsoft Purview**: Cloud-backed endpoint DLP enforcement and monitoring.

## 5. Build Plan

1. **Deploy pfSense**  
   - Create VLANs (810, 820, 830).  - (Done)
   - Configure interfaces, firewall rules, and NAT.  - (Done)
   - Ensure client → gateway → DMZ path enforcement. - (Done - Needs Documentation)

2. **Deploy Wazuh SIEM**  
   - Install Wazuh all-in-one on Ubuntu.  - (Done)
   - Register the Windows endpoint.  - (Done)
   - Configure FIM and USB monitoring. - (Done)
   - Optionally configure active-response policies. - (Will Revisit)

1. **Deploy Suricata Gateway**
   - Install Suricata in inline IPS mode.  - (Done)
   - Load EmergingThreats DLP rule sets. -  (In Progress)
   - Enable EVE JSON logging and forward logs to Wazuh (optional). - (In Progress)

4. **Configure the Windows Endpoint**  
   - Seed with synthetic PII/PCI test files.  
   - Install and enroll the Wazuh agent.  - (Done)
   - Validate FIM and USB detection. - (Done - Needs Documentation)

5. **Deploy External Server**  
   - Install Nginx, FTP, SSH, or similar services.  
   - Use as the “public” exfiltration target reachable only through pfSense routing.

## 6. Test Scenarios

- **Network Exfiltration**  
  Attempt HTTP, FTP, SCP, or other file transfers from the Windows host.  
  Suricata inspects flows and issues DLP alerts.

- **File Integrity Violations**  
  Modify, delete, or rename sensitive files.  
  Wazuh FIM generates real-time alerts.

- **USB Exfiltration Attempts**  
  Copy sensitive data to a virtual USB storage device.  
  Wazuh detects insertion and logs file activity.

- **Optional: Microsoft Purview**  
  Use Purview DLP policies to block USB writes and cloud uploads.

## 7. Deliverables

- **Architecture diagram** (PNG/SVG) showing pfSense + VLANs + traffic flow.
- **Operational runbook** (3–5 pages).
- **Evidence bundle** containing Suricata, Wazuh, and endpoint logs.
- **GitHub repository** containing reproducible configs, scripts, and documentation.
- **Final project report** summarizing design, controls, and findings.