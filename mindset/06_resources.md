# 🔗 Resources & Tools — Hacker Mindset Toolkit

> **Curated collection of resources to support your 30-day challenge and beyond.**

---

## 📚 Learning Platforms

| Platform | Type | Cost | Best For |
|----------|------|------|----------|
| [TryHackMe](https://tryhackme.com/) | Guided labs | Free + Premium | Beginners, structured paths |
| [HackTheBox](https://www.hackthebox.com/) | CTF + Machines | Free + VIP | Intermediate+, real machines |
| [PicoCTF](https://picoctf.org/) | CTF | Free | Beginners, students |
| [PortSwigger Academy](https://portswigger.net/web-security) | Web labs | Free | Web security deep dive |
| [OverTheWire](https://overthewire.org/) | Wargames | Free | Linux, command-line mastery |
| [VulnHub](https://www.vulnhub.com/) | Downloadable VMs | Free | Offline practice |
| [Hack The Box Academy](https://academy.hackthebox.com/) | Courses | Free + Paid | Structured learning modules |
| [SANS Cyber Aces](https://www.cyberaces.org/) | Courses | Free | Networking, OS fundamentals |

---

## 🔧 Essential Tools

### Reconnaissance
| Tool | Purpose | Command Example |
|------|---------|-----------------|
| **Nmap** | Port scanning & service detection | `nmap -sV -sC target_ip` |
| **WHOIS** | Domain registration info | `whois example.com` |
| **dig** | DNS queries | `dig example.com ANY` |
| **theHarvester** | Email & subdomain discovery | `theHarvester -d example.com -b google` |
| **Shodan** | Internet-connected device search | [shodan.io](https://www.shodan.io/) |
| **Wappalyzer** | Web technology detection | Browser extension |

### Web Application Testing
| Tool | Purpose | Command Example |
|------|---------|-----------------|
| **Burp Suite** | HTTP proxy & testing | GUI — intercept & modify requests |
| **OWASP ZAP** | Web app scanner (free) | GUI — automated scanning |
| **Nikto** | Web server scanner | `nikto -h http://target` |
| **Gobuster** | Directory bruteforcing | `gobuster dir -u http://target -w wordlist.txt` |
| **SQLMap** | Automated SQL injection | `sqlmap -u "http://target?id=1" --dbs` |
| **XSSer** | XSS detection | `xsser -u "http://target?q=test"` |

### Password Cracking
| Tool | Purpose | Command Example |
|------|---------|-----------------|
| **John the Ripper** | Hash cracking | `john --wordlist=rockyou.txt hash.txt` |
| **Hashcat** | GPU-accelerated cracking | `hashcat -m 0 hash.txt rockyou.txt` |
| **Hydra** | Online brute-forcing | `hydra -l admin -P passwords.txt target ssh` |
| **CeWL** | Custom wordlist generator | `cewl http://target -d 2 -m 5 -w wordlist.txt` |

### Exploitation
| Tool | Purpose | Command Example |
|------|---------|-----------------|
| **Metasploit** | Exploitation framework | `msfconsole` → `search` → `use` → `exploit` |
| **netcat** | Network Swiss army knife | `nc -lvnp 4444` (listener) |
| **curl** | HTTP requests from CLI | `curl -X POST -d "user=admin" http://target` |

### Networking & Sniffing
| Tool | Purpose | Command Example |
|------|---------|-----------------|
| **Wireshark** | Packet capture & analysis | GUI — capture and filter traffic |
| **tcpdump** | CLI packet capture | `tcpdump -i eth0 port 80` |
| **Responder** | LLMNR/NBT-NS poisoning | `responder -I eth0` |

---

## 📖 Must-Read Books

| Book | Author | Focus |
|------|--------|-------|
| *The Web Application Hacker's Handbook* | Stuttard & Pinto | Web app security bible |
| *Hacking: The Art of Exploitation* | Jon Erickson | Low-level hacking fundamentals |
| *Penetration Testing* | Georgia Weidman | Practical pentest guide |
| *The Tangled Web* | Michal Zalewski | Browser security deep dive |
| *Real-World Bug Hunting* | Peter Yaworski | Bug bounty techniques |
| *Black Hat Python* | Justin Seitz | Python for hackers |
| *Social Engineering* | Christopher Hadnagy | Human hacking |
| *RTFM: Red Team Field Manual* | Ben Clark | Quick reference commands |

---

## 🎥 YouTube Channels

| Channel | Focus |
|---------|-------|
| [NetworkChuck](https://www.youtube.com/@NetworkChuck) | Beginner-friendly hacking & networking |
| [John Hammond](https://www.youtube.com/@_JohnHammond) | CTF walkthroughs & security research |
| [IppSec](https://www.youtube.com/@ippsec) | HackTheBox machine walkthroughs |
| [LiveOverflow](https://www.youtube.com/@LiveOverflow) | Deep technical exploits & concepts |
| [The Cyber Mentor](https://www.youtube.com/@TCMSecurityAcademy) | Practical ethical hacking |
| [David Bombal](https://www.youtube.com/@davidbombal) | Networking & security labs |
| [HackerSploit](https://www.youtube.com/@HackerSploit) | Penetration testing tutorials |
| [Nahamsec](https://www.youtube.com/@NahamSec) | Bug bounty hunting |

---

## 🌐 News & Blogs

| Source | Type | URL |
|--------|------|-----|
| The Hacker News | Daily security news | [thehackernews.com](https://thehackernews.com/) |
| Krebs on Security | In-depth breach analysis | [krebsonsecurity.com](https://krebsonsecurity.com/) |
| BleepingComputer | Threat intelligence | [bleepingcomputer.com](https://www.bleepingcomputer.com/) |
| PortSwigger Daily Swig | Web security news | [portswigger.net/daily-swig](https://portswigger.net/daily-swig) |
| HackerOne Hacktivity | Real bug reports | [hackerone.com/hacktivity](https://hackerone.com/hacktivity) |
| OWASP Blog | Web security standards | [owasp.org/blog](https://owasp.org/www-community/) |

---

## 🎯 Bug Bounty Platforms

| Platform | Beginner Friendly | URL |
|----------|-------------------|-----|
| HackerOne | ✅ Yes | [hackerone.com](https://hackerone.com/) |
| Bugcrowd | ✅ Yes | [bugcrowd.com](https://www.bugcrowd.com/) |
| Intigriti | ✅ Yes | [intigriti.com](https://www.intigriti.com/) |
| Synack | ❌ Invite only | [synack.com](https://www.synack.com/) |

---

## 🏅 Certifications (Future Goals)

| Certification | Level | Focus |
|---------------|-------|-------|
| CompTIA Security+ | Beginner | Broad security foundations |
| CEH (Certified Ethical Hacker) | Intermediate | Ethical hacking methodology |
| eJPT (eLearnSecurity Junior Pentester) | Beginner | Practical pentest |
| OSCP (Offensive Security Certified Professional) | Advanced | Hands-on penetration testing |
| PNPT (Practical Network Penetration Tester) | Intermediate | Real-world pentest |
| BSCP (Burp Suite Certified Practitioner) | Intermediate | Web application testing |

---

## 💬 Communities

| Community | Platform | Focus |
|-----------|----------|-------|
| r/netsec | Reddit | Security news & research |
| r/hacking | Reddit | General hacking discussion |
| r/AskNetsec | Reddit | Q&A for security professionals |
| HackTheBox Discord | Discord | CTF collaboration |
| TryHackMe Discord | Discord | Learning & help |
| InfoSec Twitter/X | Twitter | Security researcher community |
| OWASP Local Chapters | Meetup | In-person networking |

---

## 🧪 Practice Labs Setup

### Recommended Home Lab
```
1. VirtualBox or VMware (free) — Hypervisor
2. Kali Linux — Attacker machine
3. Metasploitable 2/3 — Vulnerable Linux target
4. DVWA (Docker) — Vulnerable web app
5. OWASP Juice Shop (Docker) — Modern vulnerable web app
6. Windows VM — Windows target for Active Directory labs
```

### Quick Start with Docker
```bash
# Run DVWA
docker run --rm -it -p 80:80 vulnerables/web-dvwa

# Run Juice Shop
docker run --rm -p 3000:3000 bkimminich/juice-shop

# Run WebGoat
docker run --rm -p 8080:8080 webgoat/webgoat
```

---

> *"The best investment a hacker can make is in continuous learning. The landscape changes daily — your knowledge must keep up."*
