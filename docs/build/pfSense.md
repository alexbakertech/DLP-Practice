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

In the Memory tab, the default value of 2Gb of RAM is plenty for our use case, so you can continue without changing anything.

Next is the Network tab. When creating a new VM, Proxmox only expects us to need one network adapter per device. Since we want to set up two network interfaces, we will check the `No network device` box for now and configure our networking separately.

![](assets/Pasted%20image%2020251116114918.png)

Finally, continue to the Confirm tab, review the VM settings, and when ready, click `Finish` to complete the configuration. Proxmox will then allocate the resources for your VM and youll be one step closer to a functioning machine.

#### Network Configuration

![](assets/Pasted%20image%2020251116121011.png)