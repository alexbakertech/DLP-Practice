[Wazuh](https://wazuh.com/) is an open source security platform that will allow us to collect logs and configure alerts based on client actions.

## Installation

I installed Wazuh on an LXC container within Proxmox. The container has the following specs:

| CPU    | RAM   | Storage |
| ------ | ----- | ------- |
| 4 vCPU | 8 GiB | 50 GB   |

Installing Wazuh was straighforward with their [documentation](http://documentation.wazuh.com/current/quickstart.html). It only requires a single command to get it up and running initially:

```
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```