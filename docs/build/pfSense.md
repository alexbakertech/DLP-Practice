# pfSense Virtual Machine

The pfSense virtual machine will be the heart of our experimental network. It will route all of the traffic for the rest of the VMs and provide realistic network segments for our DLP project.

## Obtain the pfSense ISO installer

A copy of the pfSense ISO can be obtained from this [mirror](https://atxfiles.netgate.com/mirror/downloads/) server provided by Netgate. For this lab, pfSense is serving as more of a platform for our other devices so the specific version does not matter. Just download the latest available image.

Netgate provides the pfSense images in a image.iso.gz format that will need to be decompressed prior to loading it into your hypervisor.

- To decompress the file on linux, simply run 'gunzip -c pfsense.iso.gz > pfsesne.iso' 
	![](assets/gunzip-demo.png)

- To decompress the file on Windows, you can use 7zip. Simply right click on the file, select 7-Zip, and click Extract Here.  
	![](assets/7-zip-demo.png)

Finally, upload the ISO file to Proxmox (or your virtualization platform of choice)

![](assets/pfSense-iso-uploaded.png)
## Creating the VM

Virtualizing a router takes a few extra steps than most VMs, but its pretty straightforward if its just for a lab environment. Basically you will need to have at least two assignable interfaces so you can distinguish WAN and LAN traffic. For this lab, we also want the LAN side to be VLAN aware.

### Creating Interfaces for pfSense in Proxmox

Proxmox by default is managed over its own web interface and has a default bridge interface we can use to allow our pfSense VM to access the internet, but we also need to make an internal interface that will serve as the backplane of our lab network.

To manage Proxmox's network interfaces, sign in to the web UI and navigate to `Datacenter > 'servername' > System > Netowrk` and you will see a list of interfaces for your Proxmox server.

![](assets/Pasted%20image%2020251116104355.png)

#### Identify WAN bridge

Next, we want to identify our bridge interface. This will be what we assign to our pfSense VM to act as the WAN side of the router. For my setup it is `vmbr0`, and since my home network has configured VLANS, I also need to ensure that the bridge is VLAN aware, so I can put the VM on my lab network so it doesn't interfere with regular operations.


![](assets/Pasted%20image%2020251116104720.png)

Take note of the interface name and be prepared to use it later when we configure the actual pfSense VM. This interface will provide our lab network with internet access so we can download our tools later.
#### Create the test network interface

Now that we have identified the WAN connection, its time to create our internal bridge network, which will be our lab's LAN. To do this, select `Create > Linux Bridge`

![](assets/Pasted%20image%2020251116105958.png)

When you do this, you'll see a popup prompting you to enter information about the interface to be created. Sine we are planning for pfSense to manage this interface, we can leave everything blank except for the name. Standard naming convention is to use `vmbr#` where # is the interface number. In our case, this is the second bridge interface on our system, so the name will be `vmbr1`. We also want to ensure that the `Autostart` and `VLAN aware` options are both checked. Then click `Create` and note down the interface name you assigned to this bridge.

![](assets/Pasted%20image%2020251116111435.png)

Finally, click `Apply Configuration` in Proxmox's network manager to reload the networking daemon and apply the changes. Once that is done, its time to create the actual VM for pfSense.

![](assets/Pasted%20image%2020251116111901.png)
### Configuring the Virtual Machine

With the ISO file in hand and your network interfaces created and identified, its time to put it all together and install pfSense into a VM. 

#### Creation Wizard

To get started, click the `Create VM` button in the top right corner of the Proxmox web UI.

![](assets/Pasted%20image%2020251116112452.png)

A popup will appear to allow us to configure the new VM. The first page asks about general information like the VM name and VM ID. For this lab, I am leaving the VM ID at its default value and I am naming this VM `DLP-pfSense`

![](assets/Pasted%20image%2020251116113457.png)

Continuing the setup, we are prompted for the ISO image we obtained and uploaded earlier. Select the pfSense ISO file from the dropdown and leave the rest as default.

![](assets/Pasted%20image%2020251116113645.png)

Next is the System tab, where no configuration changes are needed. Simply click next to continue to the Disks tab. Here I like to reduce the disk size from the default 32GiB to 16GiB since the pfSense install itself does not take up too much space.

![](assets/Pasted%20image%2020251116114118.png)

Under the CPU tab, assign two cores and leave the rest of the configuration at their default settings.

![](assets/Pasted%20image%2020251116114321.png)

In the Memory tab, the default value of 2Gb of RAM is a little under what we need for our use case, so bump it to 4Gb (4096MiB).

Next is the Network tab. When creating a new VM, Proxmox only expects us to need one network adapter per device. Since we want to set up two network interfaces, we will check the `No network device` box for now and configure our networking separately.

![](assets/Pasted%20image%2020251116114918.png)

Finally, continue to the Confirm tab, review the VM settings, and when ready, click `Finish` to complete the configuration. Proxmox will then allocate the resources for your VM and youll be one step closer to a functioning machine.

![](assets/Pasted%20image%2020251116121011.png)
#### Network Configuration

As mentioned before, we skipped adding networking interfaces to this VM. Now its time to add those in. To do so, select the newly created VM from the management panel in Proxmox under `Datacenter > 'servername' > DLP-pfSense > Hardware`.

![](assets/Pasted%20image%2020251116143104.png)

##### Add WAN Adapter

Once in the hardware configuration panel, we can add our two interfaces. Click `Add > Network Device` and select the bridge we noted as the WAN bridge earlier. 

![](assets/Pasted%20image%2020251116143216.png)

If your home network does not use VLANs, leave the VLAN Tag blank. In my case, I use VLAN 40 for testing , so I will adjust my setup accordingly. Finally, be sure to uncheck the `Firewall` options since pfSense will be taking over that function.

![](assets/Pasted%20image%2020251116143319.png)

##### Add LAN Adapter

Adding the LAN bridge is the same process as adding the WAN bridge. Just be sure to change the Bridge interface to match the one we created earlier and leave the VLAN Tag section blank.

![](assets/Pasted%20image%2020251116143827.png)

##### Make Note of Adapter MAC Addresses

When configuring pfSense, you will be asked to assign interfaces for your WAN and LAN networks. To make this easier, note the MAC Addresses of the interfaces you created. They can be seen in the hardware tab we were just in. Just note which MAC Address goes to the WAN interface and which one goes to the LAN interface.

![](assets/Pasted%20image%2020251116151839.png)

##### Enabling and Disabling Network Adapters

At some point, you may find it useful to  disconnect and reconnect the virtual network adapters we created from the VM. On a physical machine, you would simply unplug the ethernet adapter from the corresponding ports and plug them back in.

We can simulate this by double clicking on the interface we want to unplug and checking the `Advanced` checkbox, which will reveal a `Disconnect` option we can select. When this option is enabled, the VM will present a network device that is not connected to a network. 

![](assets/Pasted%20image%2020251116145636.png)


#### Install and Configure pfSense
##### First Boot and pfSense Install

With the initial configuration complete, lets boot up the VM. Select the VM from the server view and click the start button on the top right of the screen. 

![](assets/Pasted%20image%2020251116145935.png)

Once the VM is started, switch to the Console view in the Proxmox web UI. There you will see the pfSense Installer waiting for your input.

![](assets/Pasted%20image%2020251116150204.png)

While in the Console view, hit enter to Accept the Notices, and select the Install option on the next screen.

![](assets/Pasted%20image%2020251116150321.png)

For disk partitioning, select `Auto (ZFS)` and select `Install` on the Options page. There is no need to change anything here.

![](assets/Pasted%20image%2020251116150425.png)

Select `stripe` for ZFS Virtual Device type. We do not need any sort of RAID since this is a temporary installation.

![](assets/Pasted%20image%2020251116150543.png)

Next you will need to press the spacebar to select the disk we will be installing pfSense onto. Since this is in a VM and we only configured one disk, there will be only one option. However, it is not selected by default, so select it so that you see the `*` between the two brackets, and then press enter to continue with the installation.

![](assets/Pasted%20image%2020251116150721.png)

Finally, we are asked to confirm our changes and format the disk. Select yes and press enter.

![](assets/Pasted%20image%2020251116150752.png)

Once pfSense has been installed to the disk, you will be asked to reboot. Press enter to continue.

![](assets/Pasted%20image%2020251116150942.png)

##### Initial pfSense Configuration

Once pfSense has rebooted, you will be asked if VLANs should be configured. 

![](assets/Pasted%20image%2020251116152051.png)

We want to say yes, as we are configuring the following VLANS:
- **VLAN 810** – Client Network (`win11-client`, `wazuh-siem`)
- **VLAN 820** – Security Gateway (`dlp-gateway`)
- **VLAN 830** – DMZ (`external-server`)

So enter `y` and press enter to continue.

##### VLAN Configuration

Next youll be asked to select the parent interface of your VLAN. You should select the interface that matches the mac address of the LAN Network Adapter your took note of earlier. 

For me that is `vtnet1`. Enter your parent interface name and press enter. 

Then enter the VLAN tag for the new sub interface and press enter again.

![](assets/Pasted%20image%2020251116152723.png)

pfSense will ask you these same two questions on repeat until you press enter without putting anything in as a response. This allows you to enter as many VLANs as needed. Repeat the prior two steps until you add each VLANs to the LAN Network Adapter. Then press enter to exit the prompt after you are finished.

##### WAN and LAN Configuration

After inputting all of the VLANs required, you will be asked to select the WAN interface. This will be the other interface we added for WAN access. In my case, since `vtnet1` is my LAN connection, it means that `vtnet0` is my WAN connection. Enter your WAN interface name and press enter.

You will be asked to enter your LAN interface name. This should be the same interface we set as the parent interfaces for the VLANs. For me that is `vtnet1`. Enter your LAN interface name and press enter.

Finally, youll be asked to configure any additional interfaces you may wish to add. We do not need any interfaces other than the ones we have already configured at this point, so leave this part blank and press enter to continue. And type `y` followed by enter to apply the setup.

![](assets/Pasted%20image%2020251116154046.png)

Once all of the changes are applied, you should see something like this, showing that both of your  interfaces have IP addresses assigned:

![](assets/Pasted%20image%2020251116154850.png)

### Final Note

You should now have a functioning pfSense install. I strongly recommend creating a full backup of the VM in its current state before continuing.

To do this, select the `Backup` tab in the VM management window, and click `Backup Now`. Then in the `Notes` section of the popup, enter `{{guestname}}, {{vmid}}` and click `Backup`.

![](Pasted%20image%2020251116163913.png)

Once you have backed up the VM, proceed to the [Firewall-Configuration](Firewall-Configuration.md) page to setup the firewall rules.