https://docs.netgate.com/pfsense/en/latest/development/php-shell.html#enableallowallwan

## Overview

Before starting on the steps below, please ensure you have made a backup of your pfSense VM. That way if something gets messed up, we can restore to a known working state.

This document goes over the firewall configuration changes needed to get our test network ready for our experiments.

## Enable Web UI on WAN

Since this is a lab environment and we dont have any devices on the LAN side of the pfSense VM, it becomes useful to enable access from outside networks. If you were configuring this VM as an actual router, it would be wise to skip this and instead put a device on the LAN and configure the firewall rules from there.

### Access pfSense tools

From the Console of the pfSense VM, access the terminal and type option `12` to access pfSense console tools. 

![](Pasted%20image%2020251116180311.png)

In the pfSense shell, type `enableallowallwan` and press enter to tell pfsense to add a firewall rule to allow access to the web UI from the WAN interface.

![](Pasted%20image%2020251116181005.png)

Now lets return to the main console menu by pressing `CTRL+C` and take note of the IP address on the WAN interface.

![](Pasted%20image%2020251116181330.png)

### Confirm Access to Web UI

