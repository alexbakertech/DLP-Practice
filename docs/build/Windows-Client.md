## Overview

The Windows client will simulate a standard client device on our network. It will contain simulated sensitive data that will be the target of exfiltration attempts by another device. The goal is to receive alerts when specific files are changed or copied.

## Configuration

This VM will run Windows 10 because it is easier to virtualize than Windows 11 in its current state. I have assigned the following specifications to this VM:

| CPU    | RAM   | Storage | VLAN | IPv4      |
| ------ | ----- | ------- | ---- | --------- |
| 4 vCPU | 8 GiB | 100 GB  | 810  | 10.8.10.3 |

## Install Wazuh Agent

I enrolled the Windows 10 machine with my Wazuh server via the command provided by the New Agent workflow.

![](assets/Pasted%20image%2020251123120403.png)

## Configure File Integrity Monitoring

The Wazuh agent allows for in depth monitoring of events on the target device. In this lab, we want to configure monitoring for user directories that may contain sensitive information as well as directories that may be susceptible to unauthorized access.

### Choose Directories To Monitor

I have chosen to monitor the home directories of the Administrator and the Public users:

- `C:\Users\Admin\`
- `C:\Users\Public\`

While this does not include every possible directory that is worth monitoring, these directories are sufficient for demonstration purposes.

### Backup Default Agent Configuration

