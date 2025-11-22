## Overview
[Wazuh](https://wazuh.com/) is an open source security platform that will allow us to collect logs and configure alerts based on client actions.

## Installation

I installed Wazuh on an LXC container within Proxmox. The container has the following specs:

| CPU    | RAM   | Storage |
| ------ | ----- | ------- |
| 4 vCPU | 8 GiB | 50 GB   |
- VLAN 810
  
Installing Wazuh was straightforward with their [documentation](http://documentation.wazuh.com/current/quickstart.html). It only requires a single command to get it up and running initially:

```
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

After installation, the web interface is available on port 443 of the container. For my lab, I chose to also forward the port through pfsense so I can access it from outside the lab network.

![](assets/Pasted%20image%2020251122134814.png)

Wazuh's default username is `admin` and the password is generated and output to the console during the initial installation.
## Deploying Agents

Wazuh uses agent programs to watch for changes and monitor the status of endpoints and other devices.

To deploy the agent software, you first need endpoints to deploy it to. Return to the [project index](../INDEX.md) page to read about the clients we are installing the agent to.

To deploy a new agent, sign in to Wazuh and click `Deploy new agent` on the home dashboard.

![](assets/Pasted%20image%2020251122135607.png)

Then fill out the fields  form on the new agent page.

![](assets/Pasted%20image%2020251122143703.png)

