## Initial resources

1) Existing network with VLAN 40 designated for LAB use
	1) I will use VLAN 40 as the WAN for my pfSense VM
2) Proxmox Hypervisor
	1) This instance of Proxmox is dedicated to this lab
3) Windows, Ubuntu LTS, and pfSense ISO images
	1) A Windows 10 ISO image can be created with the [Microsoft MediaCreationTool](https://support.microsoft.com/en-us/windows/create-installation-media-for-windows-99a58364-8c02-206f-aa6f-40c3b507420d#id0ejd=windows_11)
		1) I am using Windows 10 because it is easier to configure in Proxmox as a VM.
	2) An Ubuntu ISO image can be found on the official [Ubuntu website](https://ubuntu.com/download/desktop)
		1) I am using Ubuntu 24.04.3 LTS
	3) pfSense ISO image can be found [here](https://atxfiles.netgate.com/mirror/downloads/)
		1) I am using a download mirror because Netgate now requests PII to get the image from the official pfsense website.
4) Wazuh install script: 
	1) `https://packages.wazuh.com/4.8/wazuh-install.sh`
5) Emerging Threats DLP rulepack:
	1) `https://rules.emergingthreats.net/open/suricata-6.0.0/rules/emerging-dlp.rules`
6) 