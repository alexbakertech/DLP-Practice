## Overview

Suricata is an IPS that will serve as a gateway to help identify and block network anomalies and potential malicious activities on our lab network.

I initially had trouble getting Suricata to capture and forward traffic on just one network interface, but in the end I was able to have it inspect, block, and forward traffic and function as expected. This was a new setup for me since I usually deploy Suricata directly from pfSense, but this lab aims not to be tied to a specific firewall or network stack. By loading Suricata on a dedicated VM, I ensure that this setup is applicable to a wide range of network configurations.
## Configuration

| CPU    | RAM   | Storage | VLAN | IPv4      |
| ------ | ----- | ------- | ---- | --------- |
| 4 vCPU | 8 GiB | 50 GB   | 820  | 10.1.20.2 |
- I am starting with Ubuntu as most documentation assumes a Debian based distro is used. This makes troubleshooting and general OS configuration easier to figure out.
### Configure Linux VM As A Network Gatweay

For Suricata to work, our VM as an IPS, it must be able to route traffic instead of just listening passively on the network. Turning the device into a gateway will allow it to sit between client devices and the WAN interface on our firewall. This, in turn, allows Suricata to actively decide which packets should be dropped or forwarded based on rulesets we configure or download.

I enabled IP forwarding:
- `sudo sysctl -w net.ipv4.ip_forward=1`

Then configured sysctl to apply this change on every reboot:
- `echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf`

Next I confirmed that IP forwarding was enabled:
- `sudo sysctl -p`

The command above should return `1` to indicate that IP forwarding is enabled

Finally, we enable MASQUERADING for outbound traffic:
- `sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`

After this is configured, the VM is capable of routing traffic as a gateway. If you set the client device's gateway address to the IP address of this VM, traffic will pass through the VM prior to exiting via the WAN interface of the router.

### Install Suricata

Now that we have a working gateway, we can update the VM and install our monitoring software. Ill reference section 2.1 of their [official documentation](https://docs.suricata.io/en/latest/quickstart.html) for installation instructions. As of the date of writing (11/28/2025) the basic install steps are as follows:

```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata jq
```

This will result in a baseline Suricata installation that is not configured yet.

### Suricata Configuration

There are a few modifications that need to be made to the Suricata configuration file `/etc/suricata/suricata.yaml`:

First, configure the `HOME_NET` to include the subnet that our client machine is on and the subnet that the suricata vm is on:

```

```

Next is to configure the `nfq` section to set Suricata up in IPS mode instead of its default IDS mode:

```

```

Once those changes are made, we can move on to setting up our NFQUEUE firewall rules in iptables.

You can find the full configuration file here: (To be added)

### Setup iptables NFQUEUE Rules

Next is to configure broad rules that will route traffic that is inbound from our client network through the NFQUEUE that we configured through Suricata. Earlier we set the queue number to 0. This corresponds to the `--queue-num` in the commands below.

```
sudo iptables -A FORWARD -s 10.8.10.0/24 -d 0.0.0.0/0 -j NFQUEUE --queue-num 0
sudo iptables -A FORWARD -s 0.0.0.0/24 -d 10.8.10.0/24 -j NFQUEUE --queue-num 0
```

Then we want to persistently apply these rules:

```
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

With that done, we should now be routing traffic through the queue that Suricata is configured to inspect.

### Suricata Service Systemd Override

Now that our configuration is complete, we need to tell Suricata which queue to use to inspect the traffic. To do this, I changed the `ExecStart` command in the Suricata service file:

```
sudo systemctl edit --full suricata
```

Then modified `ExecStart` to be:

```
ExecStart=/usr/bin/suricata -D -c /etc/suricata/suricata.yaml -q 0 --pidfile /run/suricata.pid
```

Changed/added `After` and `Wants` dependencies

```
After=... network-online.target ...
Wants=... network-online.target ...
```

Then I reloaded and reloaded the Suricata service:

```
sudo systemctl daemon-reload
sudo systemctl restart suricata
sudo systemctl status suricata
```

### Verify Proper Operation

Now, with all of the configuration out of the way, we can confirm that Suricata is working properly. Restart the Suricata service, the run the following checks:

- Confirm queue binding:
	- `sudo tail -n 50 /var/log/suricata/suricata.log`
	- Look for `Info: nfq: binding this thread 0 to queue '0'`

- Confirm `eve.json` is open and being written to by Suricata
	- `sudo lsof /var/log/suricata/eve.json`
	- You should see that the file is being held open by the Suricata service

- Tail the alerts file (Optional)
	- `sudo tail -f /var/log/suricata/eve.json`
	- This file updates periodically with any alerts generated by Suricata

Once Suricata is confirmed working, its time to add some rulesets and ingest data to Wazuh.