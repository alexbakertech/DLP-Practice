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

I have chosen to monitor the home directories of the Administrator and the Public users and the directory that holds the Wazuh agent configuration file:

- `C:\Users\Admin\`
- `C:\Users\Public\`
- `C:\Program Files (x86)\ossec-agent\

While this does not include every possible directory that is worth monitoring, these directories are sufficient for demonstration purposes.

### Backup Default Agent Configuration

The Wazuh agent configuration is located at `C:\Program Files (x86)\ossec-agent\ossec.conf`

Before making any configuration changes, I backed up the configuration file to `ossec.conf.bkp` so I have a known working configuration on standby.

### Enable Whodata FIM In Agent Configuration File

FIM is enabled by default in the Wazuh agent, but we still need to specify our directories of interest and define the type of monitoring we want to occur.

For this lab, I want realtime monitoring of the directories mentioned above. I also want to record additional information about changes that occur. For this we will use Wazuh's whodata module.

Below is the relevant part of the configuration that I added to the `osssec.conf` file:

```
    <!-- Additional files to be monitored, manually added. -->
    <directories whodata="yes">C:\Users\Admin</directories>
    <directories whodata="yes">C:\Users\Public</directories>
    <directories whodata="yes">C:\Program%20Files%20(x86)\ossec-agent</directories>
```

### Restart Wazuh Agent

To apply the changes to the configuration file, its necessary to restart the Wazuh agent from the Windows Services Manager. 

![](Pasted%20image%2020251123144026.png)