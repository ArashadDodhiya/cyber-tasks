# 🌐 Networking Concepts — Interview Masterguide

> **Goal**: Understand every networking concept so deeply that you can explain it to a 10-year-old AND to a CISO in an interview.

---

## 1. What is a Network?

A **network** is simply two or more devices connected together to share data.

**Real-life analogy**: Think of a network like a postal system. Each house (device) has an address, and letters (data) travel through roads (cables/wireless) to reach the right house.

```
[Your PC] ----cable---- [Router] ----internet---- [Google Server]
```

### Types of Networks

| Type | Full Form | Range | Example |
|------|-----------|-------|---------|
| **PAN** | Personal Area Network | ~10 meters | Bluetooth headset to phone |
| **LAN** | Local Area Network | Building/Campus | Office network |
| **MAN** | Metropolitan Area Network | City-wide | City-wide WiFi |
| **WAN** | Wide Area Network | Country/World | The Internet |

---

## 2. OSI Model (The Backbone of Networking)

The OSI Model has **7 layers**. Think of it as **7 steps** data goes through when traveling from one device to another.

### Memory Trick: **"Please Do Not Throw Sausage Pizza Away"**

| Layer | Name | What It Does | Protocols | Devices |
|-------|------|-------------|-----------|---------|
| 7 | **Application** | What the user sees (browser, email) | HTTP, FTP, DNS, SMTP | - |
| 6 | **Presentation** | Translates data (encryption, compression) | SSL/TLS, JPEG, ASCII | - |
| 5 | **Session** | Manages connections (start, maintain, end) | NetBIOS, RPC | - |
| 4 | **Transport** | Reliable delivery, port numbers | TCP, UDP | - |
| 3 | **Network** | IP addressing, routing | IP, ICMP, ARP | Router |
| 2 | **Data Link** | MAC addressing, frame delivery | Ethernet, Wi-Fi | Switch |
| 1 | **Physical** | Actual cables, signals, bits | USB, RJ45 | Hub, Cable |

### Interview Favorite: "What happens when you type google.com?"

```
Step 1 (Application)    → Browser creates HTTP request
Step 2 (Presentation)   → Data encrypted with TLS
Step 3 (Session)        → TCP session established
Step 4 (Transport)      → Data split into segments, port 443 assigned
Step 5 (Network)        → Source & destination IP added
Step 6 (Data Link)      → MAC addresses added, frame created
Step 7 (Physical)       → Bits sent as electrical/light signals
```

---

## 3. TCP/IP Model (Practical Model)

The TCP/IP model is a **simplified 4-layer** version used in the real world.

| TCP/IP Layer | OSI Equivalent | Protocols |
|-------------|---------------|-----------|
| Application | 7, 6, 5 | HTTP, DNS, FTP |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, ARP |
| Network Access | 2, 1 | Ethernet, Wi-Fi |

---

## 4. IP Addressing

### What is an IP Address?
An IP address is like a **home address** for your device on the network.

### IPv4 (Most Common)
- Format: `192.168.1.10` (4 numbers separated by dots)
- Each number: 0-255
- Total: **32 bits** (4 octets × 8 bits)
- Total addresses: ~4.3 billion

### IPv6 (Newer)
- Format: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Total: **128 bits**
- Created because IPv4 ran out of addresses

### Public vs Private IP

| Type | Range | Who Uses It |
|------|-------|------------|
| **Private** | 10.0.0.0 – 10.255.255.255 | Home/Office LAN |
| **Private** | 172.16.0.0 – 172.31.255.255 | Medium networks |
| **Private** | 192.168.0.0 – 192.168.255.255 | Home routers |
| **Public** | Everything else | Internet-facing |

```
Example:
Your PC (Private IP: 192.168.1.5) → Router (Public IP: 203.0.113.50) → Internet
```

### Subnet Mask — What is it?
A subnet mask tells the device: **"Which part of the IP is the network, and which part is the host?"**

```
IP Address:    192.168.1.100
Subnet Mask:   255.255.255.0

Network part:  192.168.1.xxx   (identifies the network)
Host part:     xxx.xxx.xxx.100 (identifies your device)
```

### CIDR Notation
Instead of writing `255.255.255.0`, we write `/24` (because 24 bits are for the network).

| CIDR | Subnet Mask | Usable Hosts |
|------|------------|-------------|
| /8 | 255.0.0.0 | 16,777,214 |
| /16 | 255.255.0.0 | 65,534 |
| /24 | 255.255.255.0 | 254 |
| /32 | 255.255.255.255 | 1 (single host) |

---

## 5. TCP vs UDP

### TCP (Transmission Control Protocol)
- **Reliable** — guarantees delivery
- **Connection-oriented** — 3-way handshake first
- **Ordered** — data arrives in correct sequence
- Used for: HTTP, FTP, SSH, Email

### UDP (User Datagram Protocol)
- **Unreliable** — no guarantee
- **Connectionless** — just sends data
- **Fast** — no overhead
- Used for: DNS, Video streaming, Gaming, VoIP

### TCP 3-Way Handshake (Interview Must-Know!)

```
Client                    Server
  |--- SYN (Hey, can we talk?) --->|
  |<-- SYN-ACK (Sure, let's go!) --|
  |--- ACK (Great, starting!) ---->|
  |       Connection Established    |
```

### TCP 4-Way Termination

```
Client                    Server
  |--- FIN (I'm done) ----------->|
  |<-- ACK (OK, noted) -----------|
  |<-- FIN (I'm done too) --------|
  |--- ACK (OK, goodbye) -------->|
  |       Connection Closed        |
```

---

## 6. Important Ports (Must Memorize!)

| Port | Service | Protocol | What It Does |
|------|---------|----------|-------------|
| 20, 21 | FTP | TCP | File transfer |
| 22 | SSH | TCP | Secure remote access |
| 23 | Telnet | TCP | Unsecure remote access |
| 25 | SMTP | TCP | Sending emails |
| 53 | DNS | TCP/UDP | Domain name resolution |
| 67, 68 | DHCP | UDP | Auto IP assignment |
| 80 | HTTP | TCP | Web browsing (unsecure) |
| 110 | POP3 | TCP | Receiving emails |
| 143 | IMAP | TCP | Receiving emails (better) |
| 443 | HTTPS | TCP | Secure web browsing |
| 445 | SMB | TCP | File sharing (Windows) |
| 3306 | MySQL | TCP | Database |
| 3389 | RDP | TCP | Remote Desktop |
| 8080 | HTTP Alt | TCP | Alternative web server |

---

## 7. DNS (Domain Name System)

### What is DNS?
DNS is the **phonebook of the internet**. It converts human-readable names to IP addresses.

```
You type: google.com
DNS returns: 142.250.190.78
Browser connects to: 142.250.190.78
```

### DNS Resolution Process

```
1. You type "google.com" in browser
2. Browser checks its cache → Not found
3. OS checks its cache → Not found
4. Query goes to DNS Resolver (your ISP)
5. Resolver asks Root Server → "Go ask .com server"
6. Resolver asks .com TLD Server → "Go ask Google's NS"
7. Resolver asks Google's Authoritative NS → "IP is 142.250.190.78"
8. Resolver caches & returns IP to browser
```

### DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Domain → IPv4 | google.com → 142.250.190.78 |
| **AAAA** | Domain → IPv6 | google.com → 2607:f8b0::200e |
| **CNAME** | Alias/redirect | www.google.com → google.com |
| **MX** | Mail server | google.com → mail.google.com |
| **NS** | Name server | google.com → ns1.google.com |
| **TXT** | Text info | SPF, DKIM records |
| **PTR** | IP → Domain (reverse) | 142.250.190.78 → google.com |
| **SOA** | Zone authority | Primary NS info |

### DNS Security Issues (Interview Gold!)
- **DNS Spoofing/Poisoning**: Attacker injects fake DNS records → redirects you to malicious site
- **DNS Tunneling**: Hiding data inside DNS queries to bypass firewalls
- **DNS Hijacking**: Changing DNS settings to redirect traffic

---

## 8. DHCP (Dynamic Host Configuration Protocol)

DHCP **automatically assigns** IP addresses to devices on a network.

### DHCP Process (DORA)

```
D - Discover: "Hey, any DHCP server here?" (broadcast)
O - Offer:    "Yes! Here's an IP you can use: 192.168.1.50"
R - Request:  "I'll take that IP, thanks!"
A - Acknowledge: "It's yours for 24 hours (lease time)"
```

---

## 9. ARP (Address Resolution Protocol)

ARP converts **IP addresses to MAC addresses** within a local network.

```
Your PC: "Who has IP 192.168.1.1? Tell me your MAC!"
Router:  "That's me! My MAC is AA:BB:CC:DD:EE:FF"
```

### ARP Spoofing (Security Attack)
Attacker sends fake ARP replies: "I am the router!" → All traffic flows through attacker (Man-in-the-Middle).

```
Normal:  PC → Router → Internet
Attack:  PC → Attacker → Router → Internet
                ↑
         Attacker reads everything!
```

---

## 10. NAT (Network Address Translation)

NAT translates **private IPs to public IPs** so multiple devices can share one public IP.

```
Device 1 (192.168.1.2) ─┐
Device 2 (192.168.1.3) ─┤→ Router (NAT) → Internet (Public IP: 203.0.113.50)
Device 3 (192.168.1.4) ─┘
```

### Types of NAT

| Type | Description |
|------|-------------|
| **Static NAT** | One private IP → One public IP (1:1 mapping) |
| **Dynamic NAT** | Pool of public IPs shared among devices |
| **PAT (Port NAT)** | Many private IPs → One public IP (using different ports) |

---

## 11. Firewalls

A firewall is a **security guard** that monitors and controls incoming/outgoing network traffic.

### Types

| Type | Description |
|------|-------------|
| **Packet Filter** | Checks source/dest IP and port (basic) |
| **Stateful** | Tracks connection state (smarter) |
| **Application/WAF** | Inspects actual data content (smartest) |
| **Next-Gen (NGFW)** | Deep packet inspection + IDS/IPS |

### Firewall Rules Example (iptables)

```bash
# Allow SSH from specific IP
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.5 -j ACCEPT

# Block all incoming traffic by default
iptables -P INPUT DROP

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow HTTP and HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

---

## 12. VPN (Virtual Private Network)

VPN creates an **encrypted tunnel** through the internet.

```
Without VPN:
You → [ISP can see everything] → Website

With VPN:
You → [Encrypted Tunnel] → VPN Server → Website
         ISP sees gibberish
```

### VPN Protocols

| Protocol | Security | Speed | Use Case |
|----------|----------|-------|----------|
| **OpenVPN** | High | Medium | Most popular |
| **WireGuard** | High | Fast | Modern choice |
| **IPSec** | High | Medium | Corporate VPNs |
| **L2TP** | Medium | Slow | Legacy |
| **PPTP** | Low | Fast | Avoid (broken) |

---

## 13. Routing

### What is Routing?
Routing is the process of finding the **best path** for data to travel across networks.

### Static vs Dynamic Routing

| Feature | Static | Dynamic |
|---------|--------|---------|
| Configuration | Manual | Automatic |
| Best for | Small networks | Large networks |
| Adapts to changes | No | Yes |
| Protocols | None | OSPF, BGP, RIP |

### Routing Table Example

```
$ route -n
Destination     Gateway         Subnet Mask     Interface
0.0.0.0         192.168.1.1     0.0.0.0         eth0      ← Default route
192.168.1.0     0.0.0.0         255.255.255.0   eth0      ← Local network
10.0.0.0        192.168.1.254   255.255.0.0     eth0      ← Specific route
```

---

## 14. Common Network Attacks (Security Interview Must-Know!)

### Man-in-the-Middle (MITM)
Attacker secretly intercepts communication between two parties.
```
Alice → [Attacker pretending to be Bob] → Bob
```
**Prevention**: Use HTTPS, certificate pinning, VPN

### DDoS (Distributed Denial of Service)
Thousands of machines flood a server with traffic to crash it.
```
Botnet (1000s of PCs) ──→ Target Server ──→ CRASH!
```
**Prevention**: CDN, rate limiting, cloud-based DDoS protection

### DNS Spoofing
Fake DNS responses redirect users to malicious sites.
**Prevention**: DNSSEC, DNS over HTTPS (DoH)

### ARP Spoofing
Fake ARP replies redirect traffic through attacker's machine.
**Prevention**: Static ARP entries, ARP inspection, VPN

### Port Scanning (Nmap)
```bash
# Scan top 1000 ports
nmap 192.168.1.1

# Scan all ports
nmap -p- 192.168.1.1

# Service version detection
nmap -sV 192.168.1.1

# OS detection
nmap -O 192.168.1.1

# Stealth SYN scan
nmap -sS 192.168.1.1
```

---

## 15. Network Troubleshooting Commands

```bash
# Check connectivity
ping google.com

# Trace route to destination (see every hop)
traceroute google.com        # Linux
tracert google.com           # Windows

# DNS lookup
nslookup google.com
dig google.com               # Linux (more detailed)

# Check open ports on your machine
netstat -tuln                # Linux
netstat -an                  # Windows

# Check network interfaces
ifconfig                     # Linux (older)
ip addr                      # Linux (modern)
ipconfig                     # Windows

# Check ARP table
arp -a

# Check routing table
route -n                     # Linux
route print                  # Windows

# Test specific port connectivity
nc -zv 192.168.1.1 80        # Linux (netcat)
Test-NetConnection -ComputerName 192.168.1.1 -Port 80  # PowerShell
```

---

## 16. Wireless Networking (WiFi)

### WiFi Security Protocols

| Protocol | Security Level | Status |
|----------|---------------|--------|
| **WEP** | Very Weak | Cracked in minutes, NEVER use |
| **WPA** | Weak | TKIP is vulnerable |
| **WPA2** | Good | AES encryption, current standard |
| **WPA3** | Best | Latest, strongest |

### WiFi Attacks
- **Evil Twin**: Fake access point with same name → victims connect → MITM
- **Deauth Attack**: Send deauth frames → kick users off WiFi → capture handshake
- **WPS Attack**: Brute-force the 8-digit WPS PIN
- **Handshake Capture**: Capture WPA2 handshake → crack offline with wordlist

---

## 17. HTTP/HTTPS Deep Dive

### HTTP Methods

| Method | Purpose | Example |
|--------|---------|---------|
| **GET** | Retrieve data | View a webpage |
| **POST** | Send data | Submit a form |
| **PUT** | Update (full replace) | Update entire profile |
| **PATCH** | Update (partial) | Change just email |
| **DELETE** | Remove data | Delete a post |
| **OPTIONS** | Check allowed methods | CORS preflight |
| **HEAD** | GET without body | Check if page exists |

### HTTP Status Codes (Must Know!)

| Code | Meaning | Example |
|------|---------|---------|
| **200** | OK | Page loaded successfully |
| **301** | Moved Permanently | URL changed forever |
| **302** | Found (Temporary Redirect) | Login redirect |
| **400** | Bad Request | Malformed syntax |
| **401** | Unauthorized | Need to login |
| **403** | Forbidden | No permission |
| **404** | Not Found | Page doesn't exist |
| **500** | Internal Server Error | Server crashed |
| **502** | Bad Gateway | Proxy got bad response |
| **503** | Service Unavailable | Server overloaded |

### HTTPS = HTTP + TLS

```
HTTP:  Data sent in plain text → Anyone can read it
HTTPS: Data encrypted with TLS → Only sender/receiver can read
```

### TLS Handshake (Simplified)

```
Client                         Server
  |--- ClientHello (supported ciphers) --->|
  |<-- ServerHello + Certificate ----------|
  |--- Verify cert, send key exchange ---->|
  |<-- Finished --------------------------|
  |         Encrypted communication        |
```

---

## 18. Proxy Servers

### Forward Proxy
Client → **Proxy** → Internet (hides client identity)

### Reverse Proxy
Internet → **Proxy** → Server (hides server identity, load balancing)

```
Forward Proxy: You → Proxy → google.com    (Google sees proxy's IP)
Reverse Proxy: You → nginx → App Server    (You see nginx's IP)
```

---

## 19. Network Protocols Quick Reference

| Protocol | Port | Purpose |
|----------|------|---------|
| **ICMP** | - | Ping, traceroute (no port, works at IP level) |
| **SNMP** | 161/162 | Network device monitoring |
| **LDAP** | 389 | Directory services (Active Directory) |
| **LDAPS** | 636 | Secure LDAP |
| **Kerberos** | 88 | Authentication (Windows AD) |
| **NTP** | 123 | Time synchronization |
| **Syslog** | 514 | Log collection |
| **TFTP** | 69 | Simple file transfer (no auth) |

---

## 20. Top Interview Questions & Answers

### Q1: "What is the difference between TCP and UDP?"
> TCP is connection-oriented and reliable — it uses a 3-way handshake and guarantees ordered delivery. UDP is connectionless and faster but doesn't guarantee delivery. TCP is used for web, email, file transfer. UDP is used for DNS, streaming, and gaming.

### Q2: "Explain the OSI model in simple terms."
> The OSI model has 7 layers that describe how data travels from one device to another. Think of it like sending a package — Application layer is writing the letter, Transport adds tracking, Network adds the address, Data Link adds the local delivery instructions, and Physical is the actual truck carrying it.

### Q3: "What happens when you type google.com in a browser?"
> Browser checks cache → DNS resolves google.com to IP → TCP 3-way handshake → TLS handshake for HTTPS → HTTP GET request sent → Server responds with HTML → Browser renders the page.

### Q4: "How does a firewall work?"
> A firewall inspects incoming and outgoing packets against a set of rules. It can filter by IP address, port number, protocol, or even application-level content. It's like a security guard checking IDs at a building entrance.

### Q5: "What is ARP poisoning and how do you prevent it?"
> ARP poisoning is when an attacker sends fake ARP replies to associate their MAC address with the gateway's IP. This makes all traffic flow through the attacker. Prevent with static ARP entries, Dynamic ARP Inspection (DAI), and using encrypted protocols.

### Q6: "Difference between IDS and IPS?"
> **IDS** (Intrusion Detection System) only **detects** and alerts. Like a security camera.
> **IPS** (Intrusion Prevention System) **detects AND blocks**. Like a security camera with an auto-lock door.

### Q7: "What is VLAN?"
> A VLAN (Virtual LAN) logically segments a physical network into separate broadcast domains. Devices in different VLANs can't communicate directly even if on the same switch. It improves security and reduces broadcast traffic.

### Q8: "What is the difference between a hub, switch, and router?"
> **Hub**: Dumb device, sends data to ALL ports (Layer 1)
> **Switch**: Smart device, sends data only to the correct port using MAC (Layer 2)  
> **Router**: Smartest, routes data between different networks using IP (Layer 3)

---

> 💡 **Pro Tip**: In interviews, always explain concepts with analogies first, then dive into technical details. It shows you truly understand the concept, not just memorized it.

---

*Last Updated: May 2026 | Created for Cybersecurity Interview Preparation*
