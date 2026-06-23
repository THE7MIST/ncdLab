# Snort IDS Configuration on pfSense Firewall

## Objective

Install and configure Snort IDS on pfSense firewall and generate alerts for the following security events:

1. ICMP packet larger than 128 bytes.
2. More than 30 ICMP packets from a single source within 1 minute.
3. More than 20 TCP SYN packets to the web server within 1 minute.
4. SSH brute-force attack detection.
5. Kali Linux machine detection on boot.
6. Gmail access detection.
7. Facebook access detection.
8. Instagram access detection.

---

# Lab Topology

| Device           | IP Address    |
| ---------------- | ------------- |
| pfSense LAN      | 192.168.25.25 |
| Linux Web Server | 192.168.25.10 |
| Kali Linux       | 192.168.25.20 |

Network:

192.168.25.0/24

Default Gateway:

192.168.25.25

---

# Step 1: Install Apache Web Server

## CentOS / Rocky Linux

```bash
sudo yum install httpd -y

sudo systemctl enable httpd
sudo systemctl start httpd
```

Verify:

```bash
systemctl status httpd
curl localhost
```

---

# Step 2: Install Snort Package on pfSense

Navigate:

System → Package Manager → Available Packages

Search:

```
snort
```

Click:

```
Install
```

Wait until installation completes.

---

# Step 3: Enable Snort Interface

Navigate:

Services → Snort → Interfaces

Click:

```
Add
```

Select:

```
LAN
```

Enable:

* Enable Interface
* Send Alerts to System Log

Save configuration.

---

# Step 4: Configure HOME_NET

Navigate:

Services → Snort → Global Settings

Set:

```text
192.168.25.0/24
```

Save.

---

# Step 5: Enable Local Rules

Navigate:

Services → Snort → Global Settings

Enable:

```
Enable Local Rules
```

Save.

---

# Step 6: Enable local.rules Category

Navigate:

Services → Snort → Interfaces → LAN → Categories

Enable:

```
local.rules
```

Save.

---

# Step 7: Configure Local Rules

Navigate:

Services → Snort → Global Settings → Local Rules

Paste the following rules.

---

## Rule 1 – Large ICMP Packet

```snort
alert icmp any any -> $HOME_NET any \
(msg:"ICMP Packet Greater Than 128 Bytes"; \
dsize:>128; \
sid:1000001; rev:1;)
```

---

## Rule 2 – ICMP Flood Detection

```snort
alert icmp !$HOME_NET any -> $HOME_NET any \
(msg:"ICMP Flood Detected"; \
detection_filter:track by_src,count 30,seconds 60; \
sid:1000002; rev:1;)
```

---

## Rule 3 – SYN Flood Detection

```snort
alert tcp !$HOME_NET any -> 192.168.25.10 80 \
(msg:"SYN Flood Against Web Server"; \
flags:S; \
detection_filter:track by_src,count 20,seconds 60; \
sid:1000003; rev:1;)
```

---

## Rule 4 – SSH Brute Force Detection

```snort
alert tcp !$HOME_NET any -> 192.168.25.10 22 \
(msg:"SSH Brute Force Attack"; \
flags:S; \
detection_filter:track by_src,count 10,seconds 60; \
sid:1000004; rev:1;)
```

---

## Rule 5 – Kali Linux Detection

```snort
alert udp any 68 -> any 67 \
(msg:"Kali Linux Boot Detected"; \
content:"kali"; nocase; \
sid:1000005; rev:1;)
```

---

## Rule 6 – Gmail Access Detection

```snort
alert tls $HOME_NET any -> any 443 \
(msg:"GMAIL Access Detected"; \
tls.sni; content:"gmail"; nocase; \
sid:1000006; rev:1;)
```

---

## Rule 7 – Facebook Access Detection

```snort
alert tls $HOME_NET any -> any 443 \
(msg:"FACEBOOK Access Detected"; \
tls.sni; content:"facebook"; nocase; \
sid:1000007; rev:1;)
```

---

## Rule 8 – Instagram Access Detection

```snort
alert tls $HOME_NET any -> any 443 \
(msg:"INSTAGRAM Access Detected"; \
tls.sni; content:"instagram"; nocase; \
sid:1000008; rev:1;)
```

---

# Step 8: Apply Configuration

Navigate:

Services → Snort → Interfaces

Select LAN Interface

Click:

```
Save
```

Then click:

```
Restart
```

Verify status shows:

```
RUNNING
```

---

# Step 9: Generate Alerts

## Large Ping Packet

```bash
ping -s 200 192.168.25.10
```

---

## ICMP Flood

```bash
ping -f 192.168.25.10
```

---

## SYN Flood

```bash
hping3 -S -p 80 --flood 192.168.25.10
```

---

## SSH Brute Force

```bash
hydra -l root -P rockyou.txt ssh://192.168.25.10
```

---

## Kali Detection

```bash
sudo dhclient -r
sudo dhclient
```

or simply reboot Kali Linux.

---

## Gmail Detection

Open:

```text
https://gmail.com
```

---

## Facebook Detection

Open:

```text
https://facebook.com
```

---

## Instagram Detection

Open:

```text
https://instagram.com
```

---

# Step 10: Verify Alerts

Navigate:

Services → Snort → Alerts

Expected Alerts:

* ICMP Packet Greater Than 128 Bytes
* ICMP Flood Detected
* SYN Flood Against Web Server
* SSH Brute Force Attack
* Kali Linux Boot Detected
* GMAIL Access Detected
* FACEBOOK Access Detected
* INSTAGRAM Access Detected

---

# Result

Successfully installed Apache Web Server and configured Snort IDS on pfSense firewall. Custom Snort rules were created and tested successfully to detect ICMP floods, SYN floods, SSH brute-force attacks, Kali Linux hosts, and access to Gmail, Facebook, and Instagram websites.
