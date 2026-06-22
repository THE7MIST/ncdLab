# pfSense Firewall Setup Guide (VMware Workstation)

## Objective

Configure a pfSense firewall in VMware Workstation with:

* WAN connectivity through a bridged network
* LAN network: `192.168.25.0/24`
* Firewall LAN IP: `192.168.25.25`
* Client 1: `192.168.25.10`
* Client 2: `192.168.25.20`
* Internet access using NAT
* WebGUI access from the LAN network

---

## Network Topology

```text
Internet
    |
    |
192.168.1.0/24
    |
    |
WAN (em0)
pfSense Firewall
LAN (em1)
192.168.25.25/24
    |
    +-------------------+
    |                   |
    |                   |
Client 1            Client 2
192.168.25.10       192.168.25.20
GW: .25             GW: .25
```

---

## VMware Configuration

### pfSense Virtual Machine

Configure the network adapters as follows:

| Adapter           | Type            | Purpose |
| ----------------- | --------------- | ------- |
| Network Adapter 1 | Bridged         | WAN     |
| Network Adapter 2 | Custom (VMnet1) | LAN     |

```text
Network Adapter 1 = Bridged
Network Adapter 2 = VMnet1
```

---

## Reset pfSense (Optional)

If pfSense contains previous configurations, restore factory defaults.

From the pfSense console:

```text
4) Reset to factory defaults
```

Confirm:

```text
y
```

This removes:

* Firewall rules
* NAT rules
* VPN configurations
* User accounts
* Installed packages
* Interface assignments

---

## Verify Interface Assignment

From the console menu:

```text
1) Assign Interfaces
```

Verify:

```text
WAN = em0
LAN = em1
```

Expected output:

```text
WAN (wan) -> em0
LAN (lan) -> em1
```

---

## Configure WAN Interface

Select:

```text
2) Set interface(s) IP address
```

Choose WAN:

```text
1
```

Configure:

```text
Configure IPv4 address WAN interface via DHCP? (y/n) y
Configure IPv6 address WAN interface via DHCP6? (y/n) n
```

Press Enter when prompted for an IPv6 address.

Expected result:

```text
WAN -> 192.168.1.x/24
```

Example:

```text
WAN -> 192.168.1.21/24
```

---

## Configure LAN Interface

Select:

```text
2) Set interface(s) IP address
```

Choose LAN:

```text
2
```

Configure:

```text
Configure IPv4 address LAN interface via DHCP? (y/n) n
```

Enter:

```text
IPv4 Address : 192.168.25.25
Subnet Bits  : 24
Gateway      : <Press ENTER>
```

For LAN configuration:

```text
For a LAN, press ENTER for none
```

Press:

```text
ENTER
```

---

## DHCP Configuration

When prompted:

```text
Enable DHCP server on LAN?
```

Select:

```text
n
```

Static IP addressing will be used for the clients.

---

## Verify Interface Status

Expected output:

```text
WAN (wan) -> em0 -> 192.168.1.21/24
LAN (lan) -> em1 -> 192.168.25.25/24
```

---

## Configure VMnet1

Open:

```text
Edit → Virtual Network Editor
```

Select:

```text
VMnet1
```

Configure:

```text
Subnet IP   : 192.168.25.0
Subnet Mask : 255.255.255.0
```

Disable:

```text
Use local DHCP service
```

Save the configuration.

---

## Configure Client 1

Attach the client VM to:

```text
VMnet1
```

Assign:

```text
IP Address  : 192.168.25.10
Subnet Mask : 255.255.255.0
Gateway     : 192.168.25.25
DNS Server  : 8.8.8.8
```

### Linux Configuration

```bash
nmcli con mod ens33 ipv4.addresses 192.168.25.10/24
nmcli con mod ens33 ipv4.gateway 192.168.25.25
nmcli con mod ens33 ipv4.dns 8.8.8.8
nmcli con mod ens33 ipv4.method manual
nmcli con up ens33
```

---

## Configure Client 2

Assign:

```text
IP Address  : 192.168.25.20
Subnet Mask : 255.255.255.0
Gateway     : 192.168.25.25
DNS Server  : 8.8.8.8
```

---

## Connectivity Verification

### Verify LAN Connectivity

```bash
ping 192.168.25.25
```

Expected:

```text
Reply from 192.168.25.25
```

---

### Verify Internet Connectivity

```bash
ping 8.8.8.8
```

Expected:

```text
64 bytes from 8.8.8.8
```

Successful replies indicate:

* NAT is functioning
* Routing is functioning

---

### Verify DNS Resolution

```bash
ping google.com
```

Successful resolution indicates:

* DNS is functioning correctly
* Internet access is fully operational

---

## Access pfSense WebGUI

Open a browser on a client machine:

```text
https://192.168.25.25
```

Accept the certificate warning.

---

## Default Credentials

```text
Username : admin
Password : pfsense
```

---

## Verification Checklist

* [ ] WAN receives an IP address via DHCP
* [ ] LAN configured as `192.168.25.25/24`
* [ ] VMnet1 configured as `192.168.25.0/24`
* [ ] Client 1 configured as `192.168.25.10`
* [ ] Client 2 configured as `192.168.25.20`
* [ ] LAN connectivity verified
* [ ] Internet connectivity verified
* [ ] DNS resolution verified
* [ ] pfSense WebGUI accessible
* [ ] Login successful

---

## Conclusion

The pfSense firewall has been successfully configured with separate WAN and LAN interfaces. Clients on the LAN network can communicate with the firewall, access the Internet through NAT, and manage the firewall through the pfSense WebGUI.
