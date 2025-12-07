## Overview

This VM will represent any possible destination for data exfiltration from the Windows 10 Client. It will be configured with a web server that can receive file uploads and an FTP server.
## Configuration

I have assigned the following specifications to this VM:

| CPU    | RAM   | Storage | VLAN | IPv4 |
| ------ | ----- | ------- | ---- | ---- |
| 4 vCPU | 4 GiB | 50 GB   | 830  | DHCP |

I installed Kali to a VM using the [NetInstaller ISO](https://cdimage.kali.org/kali-2025.3/kali-linux-2025.3-installer-netinst-amd64.iso) image. 