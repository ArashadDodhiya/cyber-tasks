# 🔍 Phase 1: Information Gathering (Week 1-2)

> **Goal**: Master passive and active reconnaissance techniques to thoroughly enumerate targets.
> **Your existing notes**: `information_gathering/` folder

---

## 📋 Table of Contents

1. [Setup & Targets](#1-setup--targets)
2. [Passive Information Gathering](#2-passive-information-gathering)
3. [Active Scanning with Nmap](#3-active-scanning-with-nmap)
4. [Web Application Enumeration](#4-web-application-enumeration)
5. [Service Enumeration (DNS, SMB, SMTP, SNMP)](#5-service-enumeration)
6. [Vulnerability Scanning](#6-vulnerability-scanning)
7. [OSINT Tools](#7-osint-tools)
8. [Practice on Online Platforms](#8-practice-on-online-platforms)
9. [Weekly Schedule](#9-weekly-schedule)
10. [Checklist & Progress Tracker](#10-checklist--progress-tracker)

---

## 1. Setup & Targets

| Target           | IP (Example)              | What to Practice                        |
| ---------------- | ------------------------- | --------------------------------------- |
| Metasploitable 2 | 192.168.56.20             | Nmap, SMB, SMTP, vulnerability scanning |
| DVWA             | http://192.168.56.20/dvwa | Web enumeration, directory busting      |
| OWASP Juice Shop | http://localhost:3000     | Modern web app enumeration              |
| bWAPP            | http://localhost:8080     | Web vulnerability scanning              |
| scanme.nmap.org  | Online                    | Legal Nmap practice target              |

### Quick Setup

```bash
# Start Juice Shop
docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop

# Start bWAPP
docker run -d -p 8080:80 --name bwapp raesene/bwapp
# Visit http://localhost:8080/install.php first to initialize
```

---

## 2. Passive Information Gathering

> Gather information WITHOUT directly touching the target.

### 2.1 — Google Dorking

```
site:example.com filetype:pdf           # Find PDF files
site:example.com inurl:login            # Find login pages
site:example.com inurl:admin            # Find admin pages
site:example.com intitle:"index of"     # Find directory listings
site:example.com filetype:env           # Find .env files
site:example.com filetype:log "password"# Find logs with passwords
site:example.com filetype:php inurl:config  # Find PHP configs
```

> 🔗 **Google Hacking Database**: https://www.exploit-db.com/google-hacking-database

### 2.2 — WHOIS & DNS

```bash
whois example.com                    # Domain registration info
host example.com                     # Forward DNS lookup
host -t mx example.com              # Mail server records
host -t txt example.com             # TXT records (SPF/DKIM)
dig any example.com                  # All DNS records
nslookup example.com                 # Alternative DNS lookup
host 93.184.216.34                   # Reverse DNS lookup
```

### 2.3 — Shodan (https://shodan.io)

```
apache 2.2                          # Search vulnerable Apache
vsftpd 2.3.4                        # Same version as Metasploitable
port:21 country:US                   # FTP servers in US
hostname:example.com                 # Devices on a domain
vuln:CVE-2021-44228                  # Search by CVE
```

### 2.4 — theHarvester & Recon-ng

```bash
# Email & subdomain harvesting
theHarvester -d example.com -b google,bing,yahoo
theHarvester -d example.com -l 200 -b google -f output.html

# Recon-ng
recon-ng
workspaces create target_recon
marketplace install recon/domains-hosts/hackertarget
modules load recon/domains-hosts/hackertarget
options set SOURCE example.com
run
show hosts
```

---

## 3. Active Scanning with Nmap

### 3.1 — Host Discovery

```bash
nmap -sn 192.168.56.0/24             # Ping sweep
nmap -PR 192.168.56.0/24             # ARP scan (local net, fastest)
nmap -PS 192.168.56.0/24             # TCP SYN ping (bypasses ICMP block)
nmap -Pn 192.168.56.0/24             # Skip discovery, scan all hosts
```

### 3.2 — Port Scanning

```bash
sudo nmap -sS 192.168.56.20          # TCP SYN (stealth, most common)
nmap -sT 192.168.56.20               # TCP Connect (no root needed)
sudo nmap -sU --top-ports 50 192.168.56.20  # UDP scan (slow but important)
nmap -p- 192.168.56.20               # All 65535 ports
nmap -p 21,22,80,443,445 192.168.56.20  # Specific ports
nmap -F 192.168.56.20                # Fast (top 100 ports)
```

### 3.3 — Service & OS Detection

```bash
nmap -sV 192.168.56.20               # Service version detection
sudo nmap -O 192.168.56.20           # OS detection
sudo nmap -sV -sC -O 192.168.56.20   # Combined (go-to scan)
nmap -A 192.168.56.20                # Aggressive (sV+sC+O+traceroute)
```

### 3.4 — NSE Scripts

```bash
nmap -sC 192.168.56.20               # Default scripts
nmap --script vuln 192.168.56.20     # Vulnerability scan
nmap --script smb-enum-shares -p 445 192.168.56.20
nmap --script http-enum -p 80 192.168.56.20
nmap --script ftp-anon -p 21 192.168.56.20
nmap --script smtp-enum-users -p 25 192.168.56.20
```

### 3.5 — Output & Practice

```bash
nmap -sV 192.168.56.20 -oA scan_all  # Save in all formats
```

| #   | Challenge                                 | Expected Outcome       |
| --- | ----------------------------------------- | ---------------------- |
| 1   | Find all live hosts on 192.168.56.0/24    | List of IPs            |
| 2   | Find all open TCP ports on Metasploitable | ~23 open ports         |
| 3   | Identify all service versions             | Full service list      |
| 4   | Find anonymous FTP access                 | ftp-anon script result |
| 5   | Run vulnerability scan                    | Multiple CVEs found    |
| 6   | Scan `scanme.nmap.org` legally            | Compare to lab results |

---

## 4. Web Application Enumeration

### 4.1 — Directory Bruteforcing

```bash
# Gobuster
gobuster dir -u http://192.168.56.20 -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://192.168.56.20 -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak
gobuster dir -u http://localhost:3000 -w /usr/share/wordlists/dirb/common.txt

# Dirb
dirb http://192.168.56.20

# Feroxbuster (recursive, fast)
feroxbuster -u http://192.168.56.20 -w /usr/share/wordlists/dirb/common.txt
```

### 4.2 — Technology Fingerprinting

```bash
whatweb http://192.168.56.20 -v         # Identify tech stack
curl -I http://192.168.56.20            # Check response headers
nmap --script http-headers,http-title -p 80 192.168.56.20
```

### 4.3 — Nikto (Web Vulnerability Scanner)

```bash
nikto -h http://192.168.56.20           # Basic scan
nikto -h http://192.168.56.20 -p 8180   # Specific port (Tomcat)
nikto -h http://localhost:3000           # Scan Juice Shop
```

### 4.4 — Burp Suite Basics

```
1. Open Burp Suite → Proxy → Intercept On
2. Configure browser proxy: 127.0.0.1:8080
3. Browse DVWA → Inspect intercepted requests
4. Exercises:
   a) Intercept a login request — view credentials
   b) Send request to Repeater — modify & resend
   c) Check Target → Site Map for full app structure
   d) Use Intruder for parameter fuzzing
```

### 4.5 — Juice Shop Recon Challenges

Visit `http://localhost:3000/#/score-board` to track progress:

| Challenge                     | Difficulty | Hint                     |
| ----------------------------- | ---------- | ------------------------ |
| Find the Score Board          | ⭐          | Navigate to /score-board |
| Find the admin page           | ⭐          | Try /administration      |
| Access confidential document  | ⭐⭐         | Check /ftp directory     |
| Find the developer's backup   | ⭐⭐         | Look at /ftp filenames   |
| Discover hidden API endpoints | ⭐⭐         | Try /api-docs, /rest     |

---

## 5. Service Enumeration

### 5.1 — SMB Enumeration (Ports 139/445)

```bash
smbclient -L //192.168.56.20 -N        # List shares (anonymous)
smbclient //192.168.56.20/tmp -N        # Connect to share
enum4linux -a 192.168.56.20             # Full SMB enumeration
crackmapexec smb 192.168.56.20 --shares # Modern alternative
rpcclient -U "" -N 192.168.56.20        # RPC (enumdomusers, querydispinfo)
nmap --script "smb-vuln*" -p 445 192.168.56.20  # Check SMB vulns
```

### 5.2 — SMTP Enumeration (Port 25)

```bash
nc 192.168.56.20 25                     # Banner grab, then VRFY root
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.56.20
nmap --script smtp-enum-users,smtp-commands -p 25 192.168.56.20
```

### 5.3 — SNMP Enumeration (Port 161/UDP)

```bash
sudo nmap -sU -p 161 192.168.56.20     # Check if SNMP is open
snmpwalk -v 2c -c public 192.168.56.20 # Walk SNMP tree
snmp-check 192.168.56.20               # Easier formatted output
onesixtyone -c /usr/share/wordlists/SecLists/Discovery/SNMP/common-snmp-community-strings.txt 192.168.56.20
```

### 5.4 — DNS Enumeration

```bash
dnsenum example.com                     # Automated DNS enum
dnsrecon -d example.com -t std          # Standard lookup
dnsrecon -d zonetransfer.me -t axfr     # Practice zone transfer
dig axfr @nsztm1.digi.ninja zonetransfer.me  # Manual zone transfer
gobuster dns -d example.com -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## 6. Vulnerability Scanning

```bash
# Nmap vuln scripts
nmap --script vuln 192.168.56.20
nmap --script smb-vuln-ms17-010 -p 445 192.168.56.20

# Searchsploit (match service versions from Nmap)
searchsploit vsftpd 2.3.4
searchsploit apache 2.2
searchsploit samba 3.0
searchsploit -m exploits/unix/remote/17491.rb   # Copy exploit locally

# Metasploit auxiliary scanners
msfconsole
search type:auxiliary scanner
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.56.20
run
```

---

## 7. OSINT Tools

| Tool            | Purpose           | URL                             |
| --------------- | ----------------- | ------------------------------- |
| Maltego CE      | Visual OSINT      | https://maltego.com             |
| SpiderFoot      | Automated OSINT   | https://spiderfoot.net          |
| Sherlock        | Username search   | `pip3 install sherlock-project` |
| crt.sh          | Cert transparency | https://crt.sh                  |
| Wayback Machine | Archive lookup    | https://web.archive.org         |
| OSINT Framework | Tool directory    | https://osintframework.com      |

```bash
sherlock targetusername
curl "https://crt.sh/?q=%25.example.com&output=json" | jq '.[].name_value' | sort -u
```

---

## 8. Practice on Online Platforms

| Platform      | Room / Challenge       | Focus             | Time   |
| ------------- | ---------------------- | ----------------- | ------ |
| TryHackMe     | Nmap room              | Scanning          | 1 hr   |
| TryHackMe     | Passive Reconnaissance | OSINT             | 1 hr   |
| TryHackMe     | Active Reconnaissance  | Nmap/Nikto        | 1.5 hr |
| TryHackMe     | Content Discovery      | Directory enum    | 1 hr   |
| TryHackMe     | Google Dorking         | Search techniques | 1 hr   |
| TryHackMe     | Burp Suite: Basics     | Web proxy         | 1 hr   |
| OverTheWire   | Bandit levels 0-10     | Linux CLI basics  | 2 hr   |
| HackThisSite  | Basic missions 1-11    | Web recon         | 2 hr   |
| CMD Challenge | All challenges         | CLI skills        | 1 hr   |

---

## 9. Weekly Schedule

### Week 1 — Passive Recon & Nmap

| Day | Activity                              | Time  |
| --- | ------------------------------------- | ----- |
| Mon | Google Dorking + WHOIS + DNS lookups  | 2 hrs |
| Tue | theHarvester + Shodan + OSINT tools   | 2 hrs |
| Wed | Nmap: host discovery + port scanning  | 2 hrs |
| Thu | Nmap: service/version + OS detection  | 2 hrs |
| Fri | Nmap: NSE scripts + vuln scanning     | 2 hrs |
| Sat | TryHackMe: Nmap + Passive Recon rooms | 3 hrs |
| Sun | Review & document findings            | 1 hr  |

### Week 2 — Active Enumeration & Web Recon

| Day | Activity                                    | Time  |
| --- | ------------------------------------------- | ----- |
| Mon | SMB enumeration (enum4linux, smbclient)     | 2 hrs |
| Tue | SMTP + SNMP enumeration                     | 2 hrs |
| Wed | Web enum: Gobuster, Nikto, Whatweb          | 2 hrs |
| Thu | Burp Suite basics + DVWA exploration        | 2 hrs |
| Fri | Juice Shop recon challenges                 | 2 hrs |
| Sat | TryHackMe: Active Recon + Content Discovery | 3 hrs |
| Sun | OverTheWire Bandit (levels 0-10) + review   | 2 hrs |

---

## 10. Checklist & Progress Tracker

### Passive Recon
- [ ] Google Dorking (5+ dork types)
- [ ] WHOIS lookups (3+ domains)
- [ ] DNS lookups (host, dig, nslookup)
- [ ] Shodan searches (3+ queries)
- [ ] theHarvester (2+ targets)
- [ ] Certificate transparency lookups

### Active Scanning
- [ ] Nmap: Host discovery (`-sn`)
- [ ] Nmap: SYN scan (`-sS`)
- [ ] Nmap: Connect scan (`-sT`)
- [ ] Nmap: UDP scan (`-sU`)
- [ ] Nmap: Full port scan (`-p-`)
- [ ] Nmap: Version detection (`-sV`)
- [ ] Nmap: OS detection (`-O`)
- [ ] Nmap: NSE vuln scripts
- [ ] Nmap: All output formats (`-oA`)

### Web Enumeration
- [ ] Gobuster on Metasploitable, DVWA, Juice Shop
- [ ] Nikto scan on multiple targets
- [ ] Whatweb fingerprinting
- [ ] Burp Suite: intercept, Repeater, Site Map

### Service Enumeration
- [ ] SMB: smbclient + enum4linux
- [ ] SMTP: VRFY + smtp-user-enum
- [ ] SNMP: snmpwalk
- [ ] DNS: Zone transfer on zonetransfer.me

### Online Platforms
- [ ] TryHackMe: 4+ rooms completed
- [ ] OverTheWire Bandit: levels 0-10
- [ ] HackThisSite: Basic missions 1-5
- [ ] Juice Shop: Score Board found

---

> **Next Phase**: [Phase 2 — Password Attacks](../phase_2_password_attacks/password_attacks.md) 🔑
