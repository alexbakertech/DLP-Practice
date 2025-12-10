## Objective

This project aims to demonstrate common DLP practices such as File Integrity Monitoring, USB Device Insertion, and IPS network intervention. The objective is to create an environment that will alert when sensitive files are modified and when a USB device is inserted into a client device. It will prevent access to insecure services such as HTTP and FTP, which are unencrypted by default.

## Architecture Diagram

Below is the network diagram for this project:

![Architecture](https://mermaid.ink/img/pako:eNqllMtuGjEUhl_FcpRuOhAzw9WqIiFIFJRAUYcEKdCFM3MGrHhs5PGEXKW-Qxd9lO7bN-mT1HPhski6aFiAbfT95_znHPsJByoETPFCs9USTfpziezn8BD55kFAUmwDwZKkDxG6E0zWCIq4EPQAalEjAicxWt0CPfA8r1xX1jw0S-qu7p1ACaXpASHkFSV3oxSRqB6Qdyh5W6UIyI37DqX6TikIw_9VWkUJyARKqRY0WsE_pLyt1HrJDTiRkqayBr5YGnqjRDiX2670QBrNhA3gZwGK8_GpfzLyT2blITrlGtZMiE83-ui4jn7_QKNBL8l3VxfdEZroVN6ij2jaHX2llJbJ7kdRUkJguJIJirSKXw2HKpXj51zPTsSfb9-RXeYxaqTartovcuTWn9F0MKqRN0A7AHkuO8zdYP7ll0GvO-m-QXpFyP7wesd6G_a8ezF4g6sX3LRMtX_WGxcOL1e2JcDi5-y_vVL04Y4HkOSUVGgwLq9Ebms25dl16Alu25ILjrUytnIQohMZrhSXJqtwcW0KcGNs5qeaB8wwdDXM0YEUXIKN4G8Qt0QyP7NzJnjmF_mg70DnyOlkjH79RGeT4tf3zzaoV6LWzGwgDWgJBh0hn8epYFl6Ze-Lkd_zO2WP6RIpicrO7rnNqzhkki0gtn7RB3ShFklWsOvLs004u5wVGl0hKlxWPkvIcy1AbceuMLCrC3bs68NDTCMmEnBwDDpm2R4_ZapzbJY24BxTuwwhYqkwczyXL5ZbMXmtVIyp0akltUoXy61Ougqt1T5n9m2Lt6caZAi6p1JpMK23SS6C6RO-x9Qldopqba_Zargdl9RcBz9g2nSr9VazUasT0iINr9F5cfBjHpVUW16n2ejY806z3XRbHQdDyI3Sw-JVzR_Xl7_I4K4l)

pfSense serves as the firewall for the rest of our lab network. Three VLANs were configured to simulate a properly segmented enterprise network. VLAN 10 is for the client devices and our Wazuh device, VLAN 20 is the Gateway network where Suricata lives, and VLAN 30 will serve as the "Fake WAN" with our Kali VM acting as an external exfiltration destination.

The Windows 10 VM serves as  the client machine that will contain files containing PII. Wazuh serves as our SEIM and central monitoring server. Suricata serves as the inline IPS that has teh ability to detect and block suspicious unencrypted traffic that passes through it. Kali serves as the exfiltration destination.
## Key Detections

To confirm functionality, I performed the following tests to generate artifacts found in the [evidence page](evidence/Evidence.md). 

- USB Device Insertion Detection
- Modification of files configured in Wazuh's FIM module
- Unencrypted FTP exfiltration

## Lessons Learned

Overall this project was a successful step into the world of DLP and Cybersecurity. 