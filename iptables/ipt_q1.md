

Configure iptables as a network firewall. Your LAN network address is **192.168.25.0/24**. Configure two clients in the network with IP addresses **192.168.25.10** and **192.168.25.20**.

Configure the Firewall VM with LAN IP address **192.168.25.254**.

Enable IP forwarding and configure NAT such that both clients can access the Internet.

Create a new chain named **CDAC** and redirect all LAN traffic through this chain.

Configure rules in the **CDAC** chain such that:

* Client1 (**192.168.25.10**) should be allowed access only to **Microsoft**, **Oracle**, and **Google** websites.
* Client2 (**192.168.25.20**) should be allowed access only to **RedHat**, **HDFC Bank**, and **Google** websites.
* Allow DNS traffic only to and from the DNS server **192.168.72.20**.
* Block access to **Facebook**, **Instagram**, and **YouTube** for all clients.
* Log all dropped packets with a custom log prefix.
* Configure appropriate default policies so that all traffic not explicitly allowed is denied.
* Save the firewall configuration permanently and verify that all rules are retained after system reboot.


# Network Topology

```text
                    Internet
                        |
                        |
                  [ Firewall ]
           WAN: ens33   LAN: ens37
                  192.168.25.254
                        |
        --------------------------------
        |                              |
        |                              |
 Client1                        Client2
192.168.25.10              192.168.25.20
```


## 📌 Important Reality About Domain Names

### ✅ In Documentation

For readability in reports and viva discussions, you may write:

```text
google.com
microsoft.com
oracle.com
redhat.com
hdfcbank.com
facebook.com
instagram.com
youtube.com
```

This makes the firewall policy easier to understand.

---

### ⚠️ In IPTables

iptables operates at:

* Layer 3 (Network Layer)
* Layer 4 (Transport Layer)

iptables understands:

* IP Addresses
* Protocols
* Port Numbers

iptables does **NOT** understand:

* Domain Names
* URLs
* Web Pages

Example:

```bash
iptables -A CDAC -d google.com -j ACCEPT
```

This is **not reliable**.

---

### ✅ Correct Method

Resolve the domain first:

```bash
host google.com
```

or

```bash
nslookup google.com
```

Use the resulting IP address in firewall rules.

> **Viva Point:** In production environments, domain filtering is usually performed using Proxy Servers, DNS Filtering, Next-Generation Firewalls, or Web Filters rather than raw iptables rules.

---

## 🖥️ Network Assumptions

| Device             | Address        |
| ------------------ | -------------- |
| Firewall LAN       | 192.168.25.254 |
| Client1            | 192.168.25.10  |
| Client2            | 192.168.25.20  |
| DNS Server         | 192.168.72.20  |
| Internet Interface | ens33          |
| LAN Interface      | ens37          |

---

## 1️⃣ Enable IP Forwarding

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### Purpose

Linux normally behaves as a host.

This command converts Linux into a router.

### Traffic Flow

```text
Client → Firewall → Internet
```

---

## 2️⃣ Make IP Forwarding Permanent

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

### Purpose

Loads kernel parameters from configuration file.

Survives system reboot.

---

## 3️⃣ Configure NAT (Internet Sharing)

```bash
iptables -t nat -A POSTROUTING \
-o ens33 \
-j MASQUERADE
```

### Breakdown

| Option      | Meaning                           |
| ----------- | --------------------------------- |
| -t nat      | NAT Table                         |
| POSTROUTING | Packet leaving system             |
| -o ens33    | Internet Interface                |
| MASQUERADE  | Replace private IP with public IP |

### Example

Before NAT:

```text
192.168.25.10
```

After NAT:

```text
Public-IP
```

### Why Required?

Without NAT:

```text
Internet cannot send replies back to private addresses.
```

---

## 4️⃣ Allow Established Connections

```bash
iptables -A FORWARD \
-m state \
--state ESTABLISHED,RELATED \
-j ACCEPT
```

### Purpose

Allows:

* Existing Sessions
* Reply Packets
* Related Connections

### Example

```text
Client → Google
Google → Client
```

Reply traffic is automatically allowed.

---

## 5️⃣ Create Custom Chain

```bash
iptables -N CDAC
```

### Purpose

Creates a user-defined chain named:

```text
CDAC
```

Improves firewall organization.

---

## 6️⃣ Redirect LAN Traffic to CDAC

```bash
iptables -A FORWARD \
-s 192.168.25.0/24 \
-j CDAC
```

### Packet Flow

```text
FORWARD
   ↓
 CDAC
```

All LAN traffic is inspected by the CDAC chain.

---

## 7️⃣ Allow DNS Requests

```bash
iptables -A CDAC \
-s 192.168.25.0/24 \
-d 192.168.72.20 \
-p udp --dport 53 \
-j ACCEPT
```

### Breakdown

| Parameter  | Meaning      |
| ---------- | ------------ |
| -s         | Source       |
| -d         | Destination  |
| -p udp     | UDP Protocol |
| --dport 53 | DNS Port     |
| -j ACCEPT  | Allow        |

---

## 8️⃣ Allow DNS Replies

```bash
iptables -A CDAC \
-s 192.168.72.20 \
-d 192.168.25.0/24 \
-p udp --sport 53 \
-j ACCEPT
```

Allows DNS responses from the DNS server.

---

## 9️⃣ Allow TCP DNS

```bash
iptables -A CDAC \
-s 192.168.25.0/24 \
-d 192.168.72.20 \
-p tcp --dport 53 \
-j ACCEPT
```

```bash
iptables -A CDAC \
-s 192.168.72.20 \
-d 192.168.25.0/24 \
-p tcp --sport 53 \
-j ACCEPT
```

### Why?

Some DNS operations use TCP instead of UDP.

---

## 🔟 Block All Other DNS Servers

```bash
iptables -A CDAC \
-p udp --dport 53 \
-j DROP
```

```bash
iptables -A CDAC \
-p tcp --dport 53 \
-j DROP
```

### Result

Only:

```text
192.168.72.20
```

can provide DNS services.

---

## 1️⃣1️⃣ Block Facebook

```bash
iptables -A CDAC \
-d facebook.com \
-j REJECT
```

### Result

Facebook blocked for all clients.

---

## 1️⃣2️⃣ Block Instagram

```bash
iptables -A CDAC \
-d instagram.com \
-j REJECT
```

---

## 1️⃣3️⃣ Block YouTube

```bash
iptables -A CDAC \
-d youtube.com \
-j REJECT
```

---

## 1️⃣4️⃣ Allow Google for Client1

```bash
iptables -A CDAC \
-s 192.168.25.10 \
-d google.com \
-j ACCEPT
```

---

## 1️⃣5️⃣ Allow Microsoft for Client1

```bash
iptables -A CDAC \
-s 192.168.25.10 \
-d microsoft.com \
-j ACCEPT
```

---

## 1️⃣6️⃣ Allow Oracle for Client1

```bash
iptables -A CDAC \
-s 192.168.25.10 \
-d oracle.com \
-j ACCEPT
```

---

## 1️⃣7️⃣ Allow Google for Client2

```bash
iptables -A CDAC \
-s 192.168.25.20 \
-d google.com \
-j ACCEPT
```

---

## 1️⃣8️⃣ Allow RedHat for Client2

```bash
iptables -A CDAC \
-s 192.168.25.20 \
-d redhat.com \
-j ACCEPT
```

---

## 1️⃣9️⃣ Allow HDFC Bank for Client2

```bash
iptables -A CDAC \
-s 192.168.25.20 \
-d hdfcbank.com \
-j ACCEPT
```

---

## 2️⃣0️⃣ Log Dropped Packets

```bash
iptables -A CDAC \
-j LOG \
--log-prefix "CDAC_DROP: " \
--log-level 4
```

### Purpose

Logs denied packets for auditing and troubleshooting.

### Sample Log

```text
CDAC_DROP:
SRC=192.168.25.10
DST=8.8.8.8
```

View logs:

```bash
journalctl -f
```

---

## 2️⃣1️⃣ Drop Everything Else

```bash
iptables -A CDAC \
-j DROP
```

### Security Model

```text
Default Deny
```

Anything not explicitly allowed is denied.

---

## 2️⃣2️⃣ Configure Default Policies

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

### Meaning

| Chain   | Policy |
| ------- | ------ |
| INPUT   | Deny   |
| FORWARD | Deny   |
| OUTPUT  | Allow  |

---

## 2️⃣3️⃣ Verify Firewall Rules

```bash
iptables -L -n -v
```

Displays:

* Rules
* Packet Counters
* Byte Counters

---

## 2️⃣4️⃣ Verify NAT Rules

```bash
iptables -t nat -L -n -v
```

Verify presence of:

```text
POSTROUTING
MASQUERADE
```

---

## 2️⃣5️⃣ Save Configuration Permanently

Install service:

```bash
yum install iptables-services -y
```

Save rules:

```bash
iptables-save > /etc/sysconfig/iptables
```

Enable service:

```bash
systemctl enable iptables
```

Restart service:

```bash
systemctl restart iptables
```

---

## 🔄 Final Packet Flow

```text
Client
   │
   ▼
FORWARD
   │
   ▼
ESTABLISHED / RELATED
   │
   ▼
CDAC Chain
   │
   ├── DNS Allowed?
   │
   ├── Blocked Website?
   │
   ├── Allowed Website?
   │
   ▼
LOG
   ▼
DROP
   ▼
NAT (MASQUERADE)
   ▼
Internet
```
