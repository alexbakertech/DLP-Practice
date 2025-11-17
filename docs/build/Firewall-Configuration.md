## Overview

Before starting on the steps below, please ensure you have made a backup of your pfSense VM. That way if something gets messed up, we can restore to a known working state.

This document goes over the firewall configuration changes needed to get our test network ready for our experiments.

## Enable Web UI on WAN

Since this is a lab environment and we dont have any devices on the LAN side of the pfSense VM, it becomes useful to enable access from outside networks. If you were configuring this VM as an actual router, it would be wise to skip this and instead put a device on the LAN and configure the firewall rules from there.

### Access pfSense tools

From the Console of the pfSense VM, access the terminal and type option `12` to access pfSense console tools. 

![](assets/Pasted%20image%2020251116180311.png)

In the pfSense shell, type `enableallowallwan` and press enter to tell pfsense to add a firewall rule to allow access to the web UI from the WAN interface.

![](assets/Pasted%20image%2020251116181005.png)

Now lets return to the main console menu by pressing `CTRL+C` and take note of the IP address on the WAN interface.

![](assets/Pasted%20image%2020251116181330.png)

### Confirm Access to Web UI

To confirm access to the Web UI over the WAN interface, simply open a web browser on the same network as your pfSense vm and navigate to the ip address shown on the WAN interface. In my case this would be https://10.1.40.200

You will likely get a warning about an invalid certificate authority. This can be safely bypassed as its just alerting you to the fact that the ssl certificate is self issued. The site is still encrypted, but the certificate is not trusted by your web browser.

To bypass it, follow the steps displayed in your web browser.

![](assets/Pasted%20image%2020251116184442.png)

Once you have bypassed the security warning, you should see the default pfsense sign in page.

From here, you can log in with the default credentials below:
- Username: admin
- Password: pfsense

![](assets/Pasted%20image%2020251116184633.png)

Once signed in, youll be prompted to go through the setup wizard. This is optional and out of scope for this project. To bypass it, click on the pfSense logo in the top right corner of the page and you will be brough to the home screen.

![](assets/Pasted%20image%2020251116184841.png)


## Assign VLANS to Network Interfaces

