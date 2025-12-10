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

This ensures that my Admin and Public user directories are monitored. I also added the ossec-agent folder to the monitor so I know if someone tries to modify the agent configuration.
### Restart Wazuh Agent

To apply the changes to the configuration file, its necessary to restart the Wazuh agent from the Windows Services Manager. 

![](Pasted%20image%2020251123144026.png)

Once the agent is restarted, real time file monitoring will be active and logs will be viewable via the Wazuh dashboard.

## Configure USB Monitoring

Now that we are watching for changes in certain file directories, we want to configure alerts for when USB devices are connected to our Windows endpoint.

To do this, I followed [this guide](https://documentation.wazuh.com/current/user-manual/capabilities/command-monitoring/use-cases/detect-usb-storage.html) on the official Wazuh documentation page.

We modify the following config files to make this work:
- Windows Endpoint: 
	- `C:\Program Files (x86)\ossec-agent\local_internal_options.con`
- Wazuh Host: 
	- `/var/ossec/etc/shared/default/agent.conf`
	- `/var/ossec/etc/rules/local_rules.xml`

The end result is that my Wazuh dashboard generates an event every time a USB device is plugged into my Windows endpoint.

This is useful in the event that I want to take specific actions when a USB devices is inserted. Or if I want to enforce a `No-USB` policy, I can use this to record policy violations and kick off remedial action.

To validate that this rule is visible, navigate to the `Threat Hunting` page in Wazuh and filter for `rule.id = 100016`. Then plug and unplug a USB device from the VM a few times.

![](assets/Pasted%20image%2020251123175724.png)

You'll see that the logging is successful and you now have a record of USB events taking place on the Windows endpoint.

## Seeding PII In `C:\Users\Admin\Documents\`

For realistic testing, it is important to validate using mock data to ensure accurate alerts are being triggered.
### credit-card.txt
```credit-card.txt
American Express: 378282246310005  
Discover: 6011111111111117  
MasterCard: 5555555555554444  
Visa: 4111111111111111  
Visa (Luhn valid): 4242424242424242  
MasterCard (Luhn valid): 5105105105105100  
JCB: 3530111333300000
```
### ssn-list.docx
```ssn-list.docx
078-05-1120
096-13-2019
132-45-6789
219-09-9999
312-44-5555
433-77-8888
523-33-1234
612-12-1212
```
### hipaa-phi.txt
```hipaa-phi.txt
Patient Name: John Doe  
MRN: 123456789  
DOB: 01/15/1975  
SSN: 987-65-4321  
Diagnosis: Diabetes mellitus  
Medication: Metformin 500mg
```
### pci-batch.csv
```pci-batch.csv
CardNumber,Expiry,CVV,Name
4111111111111111,12/27,123,Allen Jonson
5555555555554444,09/26,456,Jane Smith
378282246310005,05/28,7899,Test User
6011111111111117,03/25,321,John Doe
```

When these files are created, Wazuh will start to track their version history. If they are modified or copied anywhere, we will get an alert. Check out my [test cases](Evidence.md) for examples of the different alerts that will be generated when data exfiltration is attempted.
