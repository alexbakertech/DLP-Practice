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

![](assets/Pasted%20image%2020251117163108.png)

The Interface Assignments screen will allow us to assign the VLANs we created earlier to interfaces that we can manage with pfSense.

Clicking the dropdown menu next to `Available network ports` wills show available networks you can add. Select `VLAN 810` that we created earlier and click `Add`. 

![](assets/Pasted%20image%2020251117163447.png)

Repeat the above step to until all of the VLANs we created earlier are assigned to their own interface, then click `Save`

![](assets/Pasted%20image%2020251117163645.png)

## Configure Network Interfaces

Now that the interfaces are assigned, its time to configure them. This involves naming them, assigning IP address ranges, and enabling them.

To get started, click on the name of the first interface that we want to modify. In my case, this will be the `OPT1` Interface

![](assets/Pasted%20image%2020251117194752.png)

On the configuration page, you will want to enable the interface, enter an appropriate name for the interface in the `Description` field, select `Static IPv4` under the `IPv4 Configuration Type` dropdown, enter an IPv4 range for the network that will live on this interface, click `Save`, and then click `Apply Changes`.

![](assets/Pasted%20image%2020251117200515.png)

Once those changes are applied, return to the `Interface Assignments` page and repeat the process for the other interfaces we created earlier.

For quick reference, here are the configurations I am using for each interface:

| VLAN | Description      | IPv4 Address |
| ---- | ---------------- | ------------ |
| 810  | Client_Network   | 10.8.10.1/24 |
| 820  | Security_Gateway | 10.8.20.1/24 |
| 830  | DMZ              | 10.8.30.1/24 |

Once the interfaces are configured, the `Interface Assignments` page should look something like this:

![](assets/Pasted%20image%2020251117202054.png)

## Enable DHCP

Some of our VMs will be using DHCP for their initial connection to our network. So navigate to `Services > DHCP Server` and configure the following for each of our created interfaces.

Check the box to enable the DHCP server, set the `Address Pool Range` to include a hand full of ip addresses. I like to include the highest 50 or so addresses, then when I assign static addresses, I use the lowest possible IP addresses that are available.

![](assets/Pasted%20image%2020251122184031.png)

Here are the address ranges I used for each interface:

| Interface        | VLAN | Address Pool Start | Address Pool End |
| ---------------- | ---- | ------------------ | ---------------- |
| CLIENT_NETWORK   | 810  | 10.8.10.200        | 10.8.10.250      |
| SECURITY_GATEWAY | 820  | 10.8.20.200        | 10.8.20.250      |
| DMZ              | 830  | 10.8.30.200        | 10.8.30.250      |

## Set Up Initial Firewall Rules

Now that we have some working interfaces, its time to add some firewall rules to allow traffic on our newly created interfaces.

To get started, navigate to `Firewall > Rules`

![](assets/Pasted%20image%2020251119201734.png)

Next, select `CLIENT_NETWORK` and click `Add`

![](assets/Pasted%20image%2020251119201938.png)

Now, we simply want to allow all traffic of any kind through this interface. If you want to, you can craft more granular firewall rules, but for our use case, allowing all traffic is useful to make sure everything works before tightening down the ruleset.

Set `Action` to Pass, `Protocol` to Any, `Source` to Any, and `Destination` to Any, then click `Save`

![](assets/Pasted%20image%2020251119202358.png)

You'll then be brought back to the rules for the selected interface. From here, we can copy the rule to the other two interfaces we created by clicking the copy button under `Actions`

![](assets/Pasted%20image%2020251119202647.png)

After clicking the copy button, simply change the `Interface` dropdown to the next interface we want to apply the rule to. In my case, this is the `Security_Gateway`. Then click `Save` once more.

![](assets/Pasted%20image%2020251119202940.png)

Repeat this step for the `DMZ` interface, then click apply when you return to the firewall rules page.

![](assets/Pasted%20image%2020251119203102.png)

This will apply all of the firewall rules we added so far. 

And with that, the lab firewall is configured. Please note that the specific rules we will end up with will likely be different from the ones we configured here. These allow all rules are a great jumping off point to make sure everything works, but as we enforce different traffic patterns, we will want to progress to a more locked down state.

The next part of this project is to configure the virtual machines to populate the network and conduct our tests. Return to the [project index](../INDEX.md) page for information on those VMs.

## Route Client VM Through Suricata Gateway

After configuring the Windows client VM, return here to see how I set up policy based routing to force the client device to route its traffic through Suricata.

The goal is for the Client's network route to look something like this:

```
Client -> Suricata -> pfSense -> WAN
```

Enforcing this path will make sure that Suricata has visibility into the client device's network activity and is in a position to reject or alert on suspicious traffic.

To do this, we want to override the default gateway in pfSense for the client device. Start by navigating to `System/Routing/Gatways` and add a new gateway. I named mine `SuricataGW` and set it to the ip address of my Suricata VM.

![](Assets/Pasted%20image%2020251130183814.png)

Be sure to save and apply changes, then navigate to `Firewall/Rules/CLIENT_NETWORK` and add a new firewall rule to the top of the rule stack. This will ensure traffic has a chance to match against it before it matches the allow all rule we configured earlier.

![](Assets/Pasted%20image%2020251130184043.png)

We want to add a `Pass` rule on the `CLIENT_NETWORK` that applies to `IPv4` addresses with `Any` protocol. The source should be the ip address of the Client VM.

![](Assets/Pasted%20image%2020251130184251.png)

Scroll down to `Extra Options` and click `Show Advanced`, then under `Advanced Options`, select the `SuricataGW` that we added earlier in the `Gateway` option.

![](Assets/Pasted%20image%2020251130184448.png)

Finally, save and apply the firewall rule like we did for the allow all rule above. You should see something like this:

![](Assets/Pasted%20image%2020251130184548.png)

This forces the client device to use Suricata as the next hop in its connection to the internet, effectively forcing Suricata to be a man in the middle and allowing it to see everything that passes from the client device and the internet.