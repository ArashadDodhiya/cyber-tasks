# 🔴 NMAP – Active Information Gathering (Command-by-Command)

---

## 1️⃣ `nmap`

```bash
nmap <target>
```

### Meaning

* Default Nmap scan
* Discovers live host
* Scans **top 1000 TCP ports**
* Uses **SYN scan** (if run as root)

---

## 2️⃣ `nmap -p 1-100`

```bash
nmap -p 1-100 <target>
```

### Meaning

* Scans **TCP ports 1 to 100**
* Faster than full scan
* Useful for quick checks

---

## 3️⃣ `nmap -sS` (SYN Scan)

```bash
nmap -sS <target>
```

### Meaning

* **TCP SYN (half-open) scan**
* Does NOT complete TCP handshake
* Fast and stealthy
* Requires **root privileges**

📌 Most commonly used scan in pentesting

---

## 4️⃣ `nmap -sT` (TCP Connect Scan)

```bash
nmap -sT <target>
```

### Meaning

* Full TCP 3-way handshake
* Used when **not root**
* Easily logged by target

---

## 5️⃣ `nmap -sU` (UDP Scan)

```bash
nmap -sU <target>
```

### Meaning

* Scans **UDP ports**
* Finds services like:

  * DNS (53)
  * SNMP (161)
  * NTP (123)
* Slow and noisy

---

## 6️⃣ `nmap -sS -sU` (SYN + UDP Scan)

```bash
nmap -sS -sU <target>
```

### Meaning

* Scans **TCP (SYN)** + **UDP** together
* Comprehensive scan
* Very noisy

---

## 7️⃣ `nmap -sn <address-range>` (Network Sweep)

```bash
nmap -sn 192.168.1.0/24
```

### Meaning

* **Host discovery only**
* No port scanning
* Finds live hosts in a network

📌 Used for **network sweeping**

---

## 8️⃣ `nmap -v -sn -oG <file>`

```bash
nmap -v -sn 192.168.1.0/24 -oG hosts.txt
```

### Meaning

* `-v` → verbose output
* `-sn` → ping scan only
* `-oG` → output in grepable format

---

### Extract live hosts

```bash
cat hosts.txt | cut -d " " -f 2
```

### Meaning

* Extracts IP addresses from output file
* Used for automation

---

## 9️⃣ `nmap -p <ports> <address-range> -oG <file>`

```bash
nmap -p 22,80,443 192.168.1.0/24 -oG scan.txt
```

### Meaning

* Scans specific ports
* Saves output for parsing

---

### Extract open ports

```bash
grep open scan.txt | cut -d " " -f 2
```

---

## 🔟 `nmap -sT -A <address-range> -oG <file>`

```bash
nmap -sT -A 192.168.1.0/24 -oG aggressive.txt
```

### Meaning

* TCP Connect scan
* Aggressive mode includes:

  * OS detection
  * Version detection
  * Script scanning
  * Traceroute

🔥 Very noisy
🔥 Easily detected

---

## 1️⃣1️⃣ `nmap -O` (OS Scan)

```bash
nmap -O <target>
```

### Meaning

* Detects operating system
* Uses TCP/IP fingerprinting
* Needs root + open ports

---

## 1️⃣2️⃣ `nmap -sV -sT -A <target>`

```bash
nmap -sV -sT -A <target>
```

### Meaning

* `-sV` → service & version detection
* `-sT` → TCP connect scan
* `-A` → aggressive scan

📌 Very detailed but **very loud**

---

## 1️⃣3️⃣ NSE Script – HTTP Headers

```bash
nmap --script http-headers <target>
```

### Meaning

* Uses **Nmap Scripting Engine**
* Fetches HTTP security headers
* Finds:

  * Server type
  * Missing security headers

---

## 1️⃣4️⃣ NSE Help

```bash
nmap --script-help
```

or

```bash
nmap --script-help http-*
```

### Meaning

* Lists available NSE scripts
* Shows script purpose and usage

---

# 🧠 Summary Table (Quick Memory)

| Command    | Purpose           |
| ---------- | ----------------- |
| `-sn`      | Host discovery    |
| `-sS`      | Stealth TCP scan  |
| `-sT`      | TCP connect scan  |
| `-sU`      | UDP scan          |
| `-sV`      | Service detection |
| `-O`       | OS detection      |
| `-A`       | Aggressive scan   |
| `-oG`      | Grepable output   | -> take output in a file
| `--script` | NSE scripts       |
