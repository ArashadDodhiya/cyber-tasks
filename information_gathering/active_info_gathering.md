# 🔴 What is Active Information Gathering?

### One-line definition

> **Active information gathering = Directly interacting with the target system to extract technical details**

You are:

* Sending packets
* Touching the target
* Generating logs
* Potentially triggering alerts

⚠️ This is **detectable**, **traceable**, and **sometimes illegal** without permission.

---

# 🆚 Passive vs Active (quick reset)

| Passive           | Active                |
| ----------------- | --------------------- |
| Whois             | Nmap                  |
| Google            | Port scanning         |
| Shodan            | Service probing       |
| No target contact | Direct target contact |
| Silent            | Noisy                 |

---

# 🧠 Why Active Recon Exists

Passive recon answers:

> *Who owns this?*

Active recon answers:

> *What is running, open, exposed, and vulnerable RIGHT NOW?*

---

# 🧪 Core Goals of Active Information Gathering

1. Discover **live hosts**
2. Identify **open ports**
3. Detect **running services**
4. Identify **versions**
5. Guess **OS**
6. Map **attack surface**

---

# 🧰 Active Information Gathering – Commands & Meaning

I’ll group them logically (this matters).

---

## 1️⃣ Host Discovery (Is the target alive?)

### 🔹 `ping`

```bash
ping example.com
```

**What it does**

* Sends ICMP Echo Request
* Waits for Echo Reply

**What you learn**

* Host is up
* Network latency
* Packet loss

**Detection**

* Logged by firewalls
* Often blocked

---

### 🔹 `fping` (faster ping)

```bash
fping -a -g 192.168.1.0/24
```

**Meaning**

* `-a` → show alive hosts
* `-g` → generate IP range

**Use**

* Network-wide host discovery

---

## 2️⃣ Port Scanning (MOST IMPORTANT)

### 🔹 `nmap` (the king 👑)

#### Basic scan

```bash
nmap example.com
```

**Means**

* Scan top 1000 TCP ports
* SYN scan (if privileged)

---

#### Specific ports

```bash
nmap -p 80,443 example.com
```

**Meaning**

* Check if HTTP/HTTPS are open

---

#### Full port scan

```bash
nmap -p- example.com
```

**Meaning**

* Scan all 65,535 TCP ports
* Very noisy

---

#### Fast scan

```bash
nmap -F example.com
```

**Meaning**

* Scan top 100 ports only

---

## 3️⃣ Service & Version Detection

### 🔹 Service scan

```bash
nmap -sV example.com
```

**What it does**

* Sends probes to open ports
* Tries to identify service + version

**Example result**

```text
80/tcp open  http Apache httpd 2.4.52
```

**Security value**

* Version = vulnerability mapping

---

## 4️⃣ OS Detection

### 🔹 OS fingerprinting

```bash
nmap -O example.com
```

**How it works**

* TCP/IP stack fingerprinting
* TTL, window size, flags

**Output**

```text
Running: Linux 5.X
```

⚠️ Needs:

* Root privileges
* Open ports

---

## 5️⃣ Aggressive Scan (One-shot recon)

### 🔹 All-in-one scan

```bash
nmap -A example.com
```

**Includes**

* OS detection
* Version detection
* Script scanning
* Traceroute

🔥 Very noisy
🔥 Very detectable

---

## 6️⃣ Banner Grabbing (Service fingerprinting)

### 🔹 `nc` (netcat)

```bash
nc example.com 80
```

Then type:

```text
HEAD / HTTP/1.1
```

**What you get**

* Server banner
* Software info

---

### 🔹 `telnet`

```bash
telnet example.com 21
```

**Used for**

* FTP / SMTP banners
* Legacy services

---

## 7️⃣ Web Server Enumeration

### 🔹 `curl`

```bash
curl -I http://example.com
```

**Meaning**

* Fetch HTTP headers only

**You learn**

* Server type
* Cookies
* Security headers

---

### 🔹 `wget`

```bash
wget http://example.com
```

**Use**

* Mirror sites
* Download exposed files

---

## 8️⃣ Directory & File Enumeration

### 🔹 `dirb`

```bash
dirb http://example.com
```

**What it does**

* Bruteforces directories
* Uses wordlists

**Finds**

* /admin
* /backup
* /uploads

---

### 🔹 `gobuster`

```bash
gobuster dir -u http://example.com -w wordlist.txt
```

**More modern & fast**

---

## 9️⃣ DNS Active Enumeration

### 🔹 `dig`

```bash
dig example.com ANY
```

**Active because**

* Direct DNS queries
* Can hit authoritative servers

---

### 🔹 Zone transfer attempt

```bash
dig axfr @ns1.example.com example.com
```

💥 If successful = **full DNS dump**
🛑 Usually blocked

---

## 🔟 Traceroute (Network path mapping)

### 🔹 `traceroute`

```bash
traceroute example.com
```

**What it reveals**

* Network hops
* Firewalls
* ISPs