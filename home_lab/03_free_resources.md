# 🌐 Free Cybersecurity Resources — Complete Guide

> **Every resource listed here is 100% free.** No credit card needed.

---

## 📋 Table of Contents

1. [Interactive Learning Platforms](#1-interactive-learning-platforms)
2. [Capture The Flag (CTF) Platforms](#2-ctf-platforms)
3. [Vulnerable Web Applications](#3-vulnerable-web-apps)
4. [Downloadable Vulnerable VMs](#4-downloadable-vulnerable-vms)
5. [Web-Based Hacking Labs](#5-web-based-hacking-labs)
6. [YouTube Channels](#6-youtube-channels)
7. [Free Courses & Reading](#7-free-courses--reading)
8. [Tools & Cheatsheets](#8-tools--cheatsheets)
9. [Practice Legally](#9-practice-legally)

---

## 1. Interactive Learning Platforms

| Platform | URL | Free Content | Best For |
|----------|-----|-------------|----------|
| **TryHackMe** | https://tryhackme.com | Many free rooms | Guided beginner learning |
| **Hack The Box** | https://hackthebox.com | Starting Point (free) | Realistic machines |
| **LetsDefend** | https://letsdefend.io | Free tier | Blue team / SOC analyst |
| **CyberDefenders** | https://cyberdefenders.org | Free challenges | Blue team / DFIR |
| **RangeForce** | https://www.rangeforce.com | Community edition | Interactive scenarios |

### TryHackMe Free Rooms (Start Here!)

| Room | Topic | Link |
|------|-------|------|
| Linux Fundamentals 1-3 | Linux basics | Search on TryHackMe |
| Intro to Networking | Networks | Search on TryHackMe |
| Nmap | Port scanning | Search on TryHackMe |
| OWASP Top 10 | Web security | Search on TryHackMe |
| Metasploit Introduction | Exploitation | Search on TryHackMe |
| Active Directory Basics | AD fundamentals | Search on TryHackMe |
| Windows PrivEsc | Windows escalation | Search on TryHackMe |
| Linux PrivEsc | Linux escalation | Search on TryHackMe |
| Burp Suite Basics | Web testing | Search on TryHackMe |

---

## 2. CTF Platforms

| Platform | URL | Difficulty | Topics |
|----------|-----|-----------|--------|
| **PicoCTF** | https://picoctf.org | Beginner | Web, forensics, crypto, RE |
| **OverTheWire** | https://overthewire.org | Beginner → Advanced | Linux, networking |
| **CTFtime** | https://ctftime.org | All levels | Aggregates all CTF events |
| **Hacker101 CTF** | https://ctf.hacker101.com | Beginner → Medium | Web hacking |
| **CryptoHack** | https://cryptohack.org | Beginner → Advanced | Cryptography |
| **pwnable.kr** | http://pwnable.kr | Medium → Hard | Binary exploitation |
| **Exploit Education** | https://exploit.education | Beginner → Advanced | Binary exploitation |

### Recommended OverTheWire Progression

```
1. Bandit      → Linux command line basics (START HERE)
2. Natas       → Web security basics
3. Leviathan   → Basic exploitation
4. Krypton     → Cryptography
5. Narnia      → Buffer overflows
```

---

## 3. Vulnerable Web Apps

Install these in your home lab for web hacking practice:

| App | Docker Command | What You Practice |
|-----|---------------|-------------------|
| **DVWA** | `docker run -d -p 80:80 vulnerables/web-dvwa` | SQLi, XSS, CSRF, File upload |
| **Juice Shop** | `docker run -d -p 3000:3000 bkimminich/juice-shop` | OWASP Top 10 (modern) |
| **WebGoat** | `docker run -d -p 8080:8080 webgoat/webgoat` | Guided web app security |
| **bWAPP** | `docker run -d -p 80:80 raesene/bwapp` | 100+ web vulnerabilities |
| **Mutillidae** | `docker run -d -p 80:80 citizenstig/nowasp` | OWASP Top 10 |
| **HackTheBox** | Manual setup | Various web challenges |

---

## 4. Downloadable Vulnerable VMs

All available at https://www.vulnhub.com

### Beginner VMs

| VM | Skills Practiced |
|----|-----------------|
| Kioptrix Level 1-5 | Enumeration, exploitation, priv esc |
| DC-1 through DC-9 | Drupal/WordPress exploitation |
| Basic Pentesting 1-2 | Full attack chain |
| Toppo | Linux priv esc |
| Bob | Linux enumeration & priv esc |

### Intermediate VMs

| VM | Skills Practiced |
|----|-----------------|
| Mr. Robot | Web enumeration, WordPress, Linux |
| Stapler | Multiple attack vectors |
| SickOs 1.1-1.2 | Web exploitation |
| Brainpan | Buffer overflow |
| PwnLab | PHP exploitation |

---

## 5. Web-Based Hacking Labs

No installation needed — practice in your browser:

| Lab | URL | Focus |
|-----|-----|-------|
| **PortSwigger Web Security Academy** | https://portswigger.net/web-security | Web app testing (BEST web resource!) |
| **SQLi Labs** | Browser-based | SQL Injection |
| **XSS Game** | https://xss-game.appspot.com | Cross-Site Scripting |
| **Google Gruyere** | https://google-gruyere.appspot.com | Web app vulnerabilities |
| **Hacker101** | https://www.hacker101.com | Web security fundamentals |
| **CMD Challenge** | https://cmdchallenge.com | Bash/Linux command line |
| **RegexCrossword** | https://regexcrossword.com | Regular expressions |

### PortSwigger Academy — Free Labs by Topic

| Topic | # of Free Labs |
|-------|---------------|
| SQL Injection | 15+ |
| Cross-Site Scripting (XSS) | 15+ |
| Authentication | 10+ |
| Directory Traversal | 5+ |
| Command Injection | 5+ |
| SSRF | 5+ |
| File Upload | 5+ |
| Access Control | 10+ |

---

## 6. YouTube Channels

| Channel | Best For | Content |
|---------|---------|---------|
| **John Hammond** | CTFs, beginner tutorials | Walkthroughs, tools |
| **IppSec** | HackTheBox walkthroughs | Detailed machine solves |
| **The Cyber Mentor** | Beginner ethical hacking | Full courses free |
| **NetworkChuck** | Networking, basics | Engaging, fun style |
| **LiveOverflow** | Binary exploitation | Deep technical content |
| **David Bombal** | Networking, pen testing | Practical demonstrations |
| **HackerSploit** | Linux, pen testing | Tutorials, tool guides |
| **Professor Messer** | Certifications | CompTIA Security+ |

### Must-Watch Free Video Courses

1. **The Cyber Mentor — Practical Ethical Hacking** (Full course on YouTube)
2. **Professor Messer — CompTIA Security+** (Full course on YouTube)
3. **John Hammond — CTF Playlist** (100+ video walkthroughs)

---

## 7. Free Courses & Reading

### Free Online Courses

| Course | Platform | Topics |
|--------|----------|--------|
| Cybersecurity Fundamentals | edX (free audit) | Basics |
| Introduction to Cyber Security | OpenLearn | Basics |
| Cisco Networking Academy | NetAcad | Networking |
| Google Cybersecurity Certificate | Coursera (audit free) | SOC, SIEM |

### Free Books & Guides

| Resource | URL |
|----------|-----|
| **OWASP Testing Guide** | https://owasp.org/www-project-web-security-testing-guide/ |
| **The Art of Hacking** | https://github.com/The-Art-of-Hacking/h4cker |
| **Hacking Articles** | https://www.hackingarticles.in/ |
| **PayloadsAllTheThings** | https://github.com/swisskyrepo/PayloadsAllTheThings |
| **HackTricks** | https://book.hacktricks.xyz/ |
| **GTFOBins** | https://gtfobins.github.io/ |
| **LOLBAS** | https://lolbas-project.github.io/ |

---

## 8. Tools & Cheatsheets

### Essential Free Tools

| Tool | Purpose | Included in Kali? |
|------|---------|-------------------|
| Nmap | Port scanning | ✅ Yes |
| Burp Suite Community | Web app testing | ✅ Yes |
| Metasploit | Exploitation framework | ✅ Yes |
| Hydra | Brute forcing | ✅ Yes |
| John the Ripper | Password cracking | ✅ Yes |
| Hashcat | Password cracking (GPU) | ✅ Yes |
| Gobuster | Directory brute forcing | ✅ Yes |
| LinPEAS/WinPEAS | Privilege escalation enum | Download separately |
| BloodHound | AD enumeration | ✅ Yes |
| Responder | LLMNR/NBT-NS poisoning | ✅ Yes |
| CrackMapExec | AD post-exploitation | ✅ Yes |
| Impacket | Windows protocol tools | ✅ Yes |

### Useful Cheatsheets

| Name | URL |
|------|-----|
| **Reverse Shell Cheatsheet** | https://www.revshells.com/ |
| **Nmap Cheatsheet** | Included in your `nmap.md` |
| **Linux PrivEsc Checklist** | https://book.hacktricks.xyz/linux-hardening/privilege-escalation |
| **Windows PrivEsc Checklist** | https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation |
| **AD Attack Cheatsheet** | https://book.hacktricks.xyz/windows-hardening/active-directory-methodology |
| **File Transfer Cheatsheet** | See your `file_transfer/10_file_transfer_cheatsheet.md` |

---

## 9. Practice Legally

> ⚠️ **Only hack systems you have explicit permission to access!**

### Platforms Where Hacking is Explicitly Authorized

- ✅ TryHackMe rooms
- ✅ Hack The Box machines
- ✅ Your own home lab VMs
- ✅ VulnHub downloaded machines
- ✅ CTF competitions
- ✅ Bug bounty programs (with scope defined):
  - HackerOne: https://www.hackerone.com
  - Bugcrowd: https://www.bugcrowd.com
  - Intigriti: https://www.intigriti.com

### Never Hack

- ❌ Systems you don't own
- ❌ Networks without written permission
- ❌ Government systems
- ❌ Other people's devices

---

**Previous**: [02_practice_plan.md](./02_practice_plan.md) | **Next**: [04_lab_cheatsheet.md](./04_lab_cheatsheet.md) 🚀
