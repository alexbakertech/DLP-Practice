## Overview

With network and file monitoring configured, it is time to test our setup and see if we can collect the information we set out to have in case of an exfiltration attempt on the seeded PII files.

I will test the following scenarios:
- USB device is inserted into Windows 10 client device
- PII files copied/moved/modified
- PII file transferred to Kali VM via FTP
- PII file submitted via POST request to Kali VM
- 