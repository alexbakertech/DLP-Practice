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

## Assign VLANS to Network Interfaces

Once signed in, youll be prompted to go through the setup wizard. This is optional and out of scope for this project. We will skip directly to configuring our network interfaces by navigating to `Interfaces > Assignemnts`

![](Pasted%20image%2020251117163108.png)

The Interface Assignments screen will allow us to assign the VLANs we created earlier to interfaces that we can manage with pfSense.

Clicking the dropdown menu next to `Available network ports` wills show available networks you can add. Select `VLAN 810` that we created earlier and click `Add`. 

![](Pasted%20image%2020251117163447.png)

Repeat the above step to until all of the VLANs we created earlier are assigned to their own interface, then click `Save`

![](Pasted%20image%2020251117163645.png)

## Configure Network Interfaces