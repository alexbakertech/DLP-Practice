## Overview

With network and file monitoring configured, it is time to test our setup and see if we can collect the information we set out to have in case of an exfiltration attempt on the seeded PII files.

I will test the following scenarios:
- USB device is inserted into Windows 10 client device
- PII files copied/moved/modified
- PII file transferred to Kali VM via FTP
- PII file submitted via POST request to Kali VM

## USB Device Insertion

When inserting a USB device to the Windows 10 Client machine, we get an alert in Wazuh:

![](assets/Pasted%20image%2020251210134544.png)

## PII File Modification

When files are modified, the `Last modified` field is updated and details about the change can be seen in the FIM Inventory tab.

![](assets/Pasted%20image%2020251210135157.png)

Details such as hash values, metadata, user IDs, etc are captured and stored for comparison to detect modification.

![](assets/Pasted%20image%2020251210135317.png)

## FTP and HTTP Exfiltration Attempts

Since we are routing the traffic of our Windows10 machine through Suricata, we are able to block insecure or suspicious file transfers. To test the effectiveness of Suricata as an IPS, I loaded up an HTTP and FTP server using the default gateway (pfSense). Then I switched the gateway to the Suricata VM and loaded the same insecure resources with the expectation that they would be blocked.

### Before Suricata

Insecure web services are allowed to load, and connections to unencrypted FTP servers are allowed.

Apache2 webserver is accessible:
![](assets/Pasted%20image%2020251210144038.png)

FTP server is accessible:
![](Pasted%20image%2020251210144111.png)

### After Suricata

Suricata now blocks access to FTP and HTTP servers, preventing easy exfiltration. Rules can be further tuned to eliminate specific threats.

Apache2 is no longer accessible:
![](Pasted%20image%2020251210144209.png)

FTP server is no longer accessible:
![](Pasted%20image%2020251210144152.png)

### A note about Suricata

Suricata is highly configurable, and it is easy to accidentally block more traffic than intended. If this is to be deployed, it may be better to clear out the ruleset and create rules specific to the deployment environment.