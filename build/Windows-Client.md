## Overview

The Windows client will simulate a standard client device on our network. It will contain simulated sensitive data that will be the target of exfiltration attempts by another device. The goal is to receive alerts when specific files are changed or copied.

## Configuration

This VM will run Windows 10 because it is easier to virtualize than Windows 11 in its current state. I have assigned the following specifications to this VM:

| CPU    | RAM   | Storage | VLAN | IPv4      |
| ------ | ----- | ------- | ---- | --------- |
| 4 vCPU | 8 GiB | 100 GB  | 810  | 10.8.10.3 |
