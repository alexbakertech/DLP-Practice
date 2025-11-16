The pfSense virtual machine will be the heart of our experimental network. It will route all of the traffic for the rest of the VMs and provide realistic network segments for our DLP project.

## Obtain the pfSense ISO installer

- A copy of the pfSense ISO can be obtained from this [mirror](https://atxfiles.netgate.com/mirror/downloads/) server provided by Netgate. For this lab, pfSense is serving as more of a platform for our other devices so the specific version does not matter. Just download the latest available image.

- Netgate provides the pfSense images in a image.iso.gz format that will need to be decompressed prior to loading it into your hypervisor.

	- To decompress the file on linux, simply run 'gunzip -c pfsense.iso.gz > pfsesne.iso' ![](gunzip-demo.png)
	
	- To decompress the file on Windows, you can use 7zip. Simply right click on the file, select 7-Zip, and click Extract Here.  ![docs/build/assets/7-zip-demo.png](7-zip-demo.png)
- Finally, upload the ISO file to Proxmox (or your virtualization platform of choice)![](pfSense-iso-uploaded.png)