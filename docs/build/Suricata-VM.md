## Overview

Suricata is an IPS that will serve as a gateway to help identify and block network anomalies and potential malicious activities on our lab network.

## Configurations


| CPU    | RAM   | Storage | VLAN | IPv4 |
| ------ | ----- | ------- | ---- | ---- |
| 4 vCPU | 8 GiB | 50 GB   | 820  |      |
  

- VLAN 820
- Assign a static IP via pfSense
- Load DLP/ET rule sets
- Configure for inline IPS mode.