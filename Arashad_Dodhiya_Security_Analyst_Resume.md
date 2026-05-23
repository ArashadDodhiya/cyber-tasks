# ARASHAD DODHIYA

**Aspiring Security Analyst | Penetration Tester**

📧 [your.email@example.com] | 📱 [+91-XXXXXXXXXX] | 📍 [City, Gujarat, India]
🔗 [LinkedIn: linkedin.com/in/arashad-dodhiya] | 💻 [GitHub: github.com/ArashadDodhiya]

---

## PROFESSIONAL SUMMARY

Motivated and detail-oriented cybersecurity enthusiast with strong hands-on experience in penetration testing, vulnerability assessment, and security analysis through extensive self-driven lab work and structured practice. Proficient in executing full penetration testing engagements — from reconnaissance and enumeration through exploitation, privilege escalation, lateral movement, and professional report writing. Skilled in web application security testing (OWASP Top 10), Active Directory attack chain execution (initial access → Domain Admin compromise), and privilege escalation on both Linux and Windows systems. Built and maintained isolated multi-machine home lab environments to simulate real-world attack scenarios end-to-end. Strong analytical mindset with a systematic, methodology-driven approach to identifying, exploiting, and documenting security vulnerabilities. Eager to apply offensive security skills in a professional penetration testing or security analyst role to help organizations identify and remediate critical vulnerabilities.

---

## TECHNICAL SKILLS

### Penetration Testing & Offensive Security
- **Reconnaissance & OSINT**: Passive and active information gathering, subdomain enumeration, Google dorking, WHOIS, DNS reconnaissance, certificate transparency logs
- **Network Scanning & Enumeration**: Port scanning, service enumeration, SMB/SMTP/SNMP enumeration, OS fingerprinting
- **Web Application Security**: SQL Injection (Union, Blind, Error-based), Cross-Site Scripting (Reflected, Stored, DOM), IDOR, SSRF, File Inclusion (LFI/RFI), File Upload Bypass, Command Injection, CSRF, SSTI, Authentication & Session Flaws, Access Control Bypass, WAF Bypass Techniques
- **Password Attacks**: Hash cracking (NTLM, MD5, SHA), brute forcing, password spraying, credential stuffing, dictionary attacks
- **Privilege Escalation (Linux)**: SUID/SGID abuse, sudo misconfigurations, cron job exploitation, weak file permissions, kernel exploits, automated enumeration (LinPEAS)
- **Privilege Escalation (Windows)**: Service permission abuse, DLL hijacking, unquoted service paths, scheduled task exploitation, token impersonation (SeImpersonate, SeBackup), UAC bypass, PowerShell history mining
- **Active Directory Attacks**: AD enumeration (PowerView, BloodHound, SharpHound), Kerberoasting, AS-REP Roasting, Pass-the-Hash, Pass-the-Ticket, DCSync, Golden/Silver Ticket attacks, LLMNR/NBT-NS poisoning, GPO abuse, lateral movement (PsExec, WMI, WinRM, RDP), credential dumping (Mimikatz, Secretsdump)
- **Network Pivoting & Tunneling**: SSH tunneling (local/remote/dynamic), Chisel, Ligolo-ng, ProxyChains, SOCKS proxying, sshuttle, double pivoting, DNS/ICMP tunneling, port redirection
- **File Transfer Techniques**: HTTP, SMB, SCP/SFTP/SSH, FTP/TFTP, Netcat, PowerShell download cradles, Base64 encoding, LOLBins-based transfers, data exfiltration methods

### Security Tools
- **Scanning & Enumeration**: Nmap, Nikto, Gobuster, DNSRecon, DNSEnum, enum4linux, Wappalyzer
- **Exploitation**: Metasploit Framework, Burp Suite, Hydra, Hashcat, John the Ripper, Searchsploit
- **Active Directory**: BloodHound, SharpHound, PowerView, CrackMapExec, Mimikatz, Impacket Suite (psexec.py, wmiexec.py, secretsdump.py, GetNPUsers.py, GetUserSPNs.py), Evil-WinRM, Responder
- **Pivoting & Tunneling**: Chisel, Ligolo-ng, ProxyChains, sshuttle, SSH, plink
- **Web Testing**: Burp Suite (Proxy, Repeater, Intruder, Decoder), Browser DevTools, cURL
- **Other**: Wireshark, Netcat, Docker, VirtualBox/VMware

### Operating Systems & Platforms
- **Linux**: Kali Linux, Ubuntu (command-line proficiency, scripting, system internals)
- **Windows**: Windows 10/11, Windows Server 2019/2022 (Active Directory, PowerShell, Group Policy)

### Programming & Scripting
- **Scripting**: Bash, PowerShell, Python (for security automation)

---

## CYBERSECURITY PROJECTS & LAB EXPERIENCE

### 🔬 Home Penetration Testing Lab — Design, Build & Attack Simulation
*Self-Initiated | Ongoing*

- Designed and built an **isolated virtualized penetration testing lab** using VirtualBox with network segmentation (NAT + Internal Network architecture) to safely practice real-world attack scenarios
- Configured **Kali Linux** as the primary attack platform alongside **Metasploitable 2** and **DVWA** as vulnerable targets on an isolated internal network (`hacklab`)
- Implemented proper **network isolation and safety protocols** — vulnerable machines completely air-gapped from host and internet to prevent accidental exposure
- Practiced **full penetration testing lifecycle**: reconnaissance → vulnerability analysis → exploitation → post-exploitation → privilege escalation → reporting
- Documented all attack methodologies, tools used, and findings in structured technical write-ups

---

### 🏰 Active Directory Attack Lab — Full Domain Compromise Simulation
*Self-Initiated | Ongoing*

- Built a **multi-machine Active Directory lab** (Windows Server 2019 Domain Controller, 2 Windows 10 workstations, Member Server, Kali attacker) on an isolated NAT network for practicing enterprise attack simulations
- Configured the domain (`corp.local`) with realistic misconfigurations: WDigest enabled, SPNs set for Kerberoasting, pre-auth disabled for AS-REP Roasting, cached Domain Admin credentials, weak local admin passwords
- Executed **end-to-end Active Directory attack chains**:
  - **Initial Access**: LLMNR/NBT-NS poisoning with Responder, password spraying
  - **Enumeration**: BloodHound/SharpHound for attack path visualization, PowerView for domain enumeration, CrackMapExec for network-wide credential testing
  - **Credential Harvesting**: Mimikatz (sekurlsa::logonpasswords), Secretsdump for remote credential dumping, Kerberoasting & AS-REP Roasting for offline hash cracking
  - **Lateral Movement**: Pass-the-Hash via CrackMapExec & Impacket PsExec, WMI execution, Evil-WinRM, RDP with stolen credentials
  - **Domain Escalation**: DCSync attack for full domain hash dump, Golden/Silver Ticket creation for persistence, GPO abuse for domain-wide control
- Studied **blue team detection** methods — mapped attacks to Windows Event IDs (4624, 4625, 4648, 4672, 7045) and understood detection tools (Sysmon, Splunk, Microsoft Defender for Identity)

---

### 🌐 Web Application Penetration Testing — DVWA & OWASP Juice Shop
*Self-Initiated | Ongoing*

- Systematically tested **DVWA at all security levels** (Low → Medium → High) across 10 vulnerability categories: Brute Force, Command Injection, CSRF, File Inclusion (LFI/RFI), File Upload, SQL Injection, Blind SQL Injection, Reflected XSS, Stored XSS, DOM-based XSS
- Practiced **filter bypass techniques** at each DVWA security level — progressing from basic payloads to advanced evasion (case variation, double encoding, comment injection, alternative tags, magic byte injection for file upload bypass)
- Exploited **OWASP Juice Shop** vulnerabilities including admin login bypass via SQLi, hidden file discovery (/ftp/ directory), DOM XSS in search functionality, and API endpoint enumeration
- Developed a deep understanding of the **web pentesting mindset** — systematic methodology covering trust boundary analysis, attack surface mapping, data flow tracing, state machine testing, and permission matrix validation
- Created comprehensive documentation covering **16 vulnerability scenario guides** with model answers, WAF bypass encyclopedia, thinking process templates, and rapid-fire cheatsheets for interview preparation

---

### 🐧 Linux Privilege Escalation — Metasploitable & Custom Labs
*Self-Initiated | Ongoing*

- Practiced **9 distinct Linux privilege escalation vectors** in isolated lab environments:
  - SUID/SGID binary abuse (GTFOBins methodology)
  - Sudo misconfigurations and sudo rule exploitation
  - Cron job hijacking and writable script exploitation
  - Weak file permissions (/etc/passwd, /etc/shadow writable)
  - Abusing authentication mechanisms
  - Kernel exploit identification and execution
- Used **automated enumeration tools** (LinPEAS, LinEnum) alongside manual enumeration for comprehensive coverage
- Documented each escalation path with step-by-step commands, explanations, and remediation recommendations

---

### 🪟 Windows Privilege Escalation — 14 Attack Vectors Documented
*Self-Initiated | Ongoing*

- Studied and practiced **14 Windows privilege escalation techniques**:
  - Situational awareness and information gathering (systeminfo, whoami /priv, netstat)
  - Service permission exploitation (weak DACLs, unquoted service paths)
  - DLL hijacking (missing DLL identification, malicious DLL placement)
  - Scheduled task abuse and writable task exploitation
  - SeImpersonate privilege abuse (PrintSpoofer, JuicyPotato)
  - SeBackupPrivilege exploitation for SAM/SYSTEM hive extraction
  - UAC bypass techniques
  - PowerShell history and transcript mining for credential discovery
  - Hidden credentials in files, registry, and environment variables
- Used tools: WinPEAS, PowerUp, Seatbelt, SharpUp for automated enumeration

---

### 🔄 Network Pivoting & Tunneling — Multi-Network Attack Simulation
*Self-Initiated | Ongoing*

- Built comprehensive knowledge of **10 pivoting and tunneling techniques** with detailed lab documentation:
  - SSH tunneling (local, remote, dynamic port forwarding)
  - Chisel (reverse SOCKS proxy, port forwarding)
  - Ligolo-ng (modern TUN-based pivoting)
  - ProxyChains + SOCKS proxies for tool routing
  - sshuttle for transparent VPN-like tunneling
  - Windows-specific tunneling (plink, netsh)
  - Double pivoting through multiple network segments
  - DNS and ICMP tunneling for restrictive environments
- Practiced **multi-machine pivot scenarios**: External → DMZ → Internal Network → Domain Controller

---

### 🔐 File Transfer & Data Exfiltration Techniques
*Self-Initiated | Ongoing*

- Documented and practiced **10 file transfer methodologies** for both Linux and Windows targets:
  - HTTP transfers (Python SimpleHTTPServer, wget, curl, PowerShell download cradles)
  - SMB file sharing (Impacket smbserver, net use)
  - SSH/SCP/SFTP secure transfers
  - FTP/TFTP transfers
  - Netcat raw socket transfers
  - Base64 encoding for inline transfers
  - LOLBins (Living-off-the-Land Binaries) — certutil, bitsadmin, mshta
  - Stealth transfer and exfiltration techniques

---

## 30-DAY HACKER MINDSET CHALLENGE

*Completed a structured 30-day self-study program focused on developing an offensive security thinking framework:*

- **Week 1 — Observe**: Learned to identify attack surfaces in everyday applications and systems
- **Week 2 — Question**: Developed the "but what if..." instinct — questioning trust boundaries, assumptions, and edge cases in software
- **Week 3 — Exploit**: Applied creative thinking to break functionality — logic bugs, second-order vulnerabilities, race conditions
- **Week 4 — Master**: Applied attacker mindset in real-world lab scenarios with full documentation
- Built daily habits: reading HackerOne disclosed vulnerability reports, analyzing CVEs, practicing on PortSwigger Academy and TryHackMe

---

## STRUCTURED LEARNING CURRICULUM

*Completed an 8-phase, 14+ week structured offensive security practice plan with hands-on lab exercises:*

| Phase | Topic | Key Skills Practiced |
|-------|-------|---------------------|
| Phase 1 | Information Gathering | Passive/active recon, Nmap scanning, SMB/SMTP enumeration |
| Phase 2 | Password Attacks | Hash cracking (Hashcat, John), brute forcing (Hydra), password spraying |
| Phase 3 | File Transfer | 10+ transfer methods across Linux and Windows environments |
| Phase 4 | Linux Privilege Escalation | SUID, sudo, cron, kernel exploits, weak permissions |
| Phase 5 | Windows Privilege Escalation | Services, DLL hijacking, tokens, UAC bypass, scheduled tasks |
| Phase 6 | Active Directory Attacks | Full AD attack chain — enumeration to Domain Admin compromise |
| Phase 7 | Port Redirection & Tunneling | SSH tunneling, Chisel, Ligolo-ng, double pivoting, ProxyChains |
| Phase 8 | Full Attack Chains | End-to-end penetration tests with professional report writing |

---

## PRACTICE PLATFORMS & CTF EXPERIENCE

| Platform | Focus Area |
|----------|-----------|
| **DVWA** | Web application vulnerabilities (all security levels) |
| **OWASP Juice Shop** | Modern web app exploitation |
| **Metasploitable 2** | Network service exploitation, multiple attack paths |
| **PortSwigger Web Security Academy** | SQL Injection, XSS, Authentication, SSRF, File Upload labs |
| **TryHackMe** | Guided offensive security rooms and learning paths |
| **Hack The Box** | Unguided machine exploitation |
| **VulnHub** | Downloadable vulnerable VMs — full compromise walkthroughs |
| **OverTheWire (Bandit)** | Linux command-line and privilege escalation wargames |

---

## EDUCATION

**[Your Degree — e.g., Bachelor of Engineering in Computer Science / Information Technology]**
*[Your University Name], [City]*
*[Graduation Year — e.g., 2025 / 2026]*

**Relevant Coursework**: Computer Networks, Operating Systems, Database Management, Data Structures, Web Technologies

---

## CERTIFICATIONS (Planned / In Progress)

- [ ] **CompTIA Security+** — Foundational security certification *(Planned)*
- [ ] **eJPT (eLearnSecurity Junior Penetration Tester)** — Practical penetration testing *(Planned)*
- [ ] **OSCP (Offensive Security Certified Professional)** — Advanced penetration testing *(Target)*
- [ ] **CRTP (Certified Red Team Professional)** — Active Directory attack & defense *(Target)*

*Update this section as you earn certifications — remove the checkboxes and add dates.*

---

## KEY STRENGTHS

- **Systematic Methodology**: Follow structured Recon → Enum → Exploit → Post-Exploit → Report methodology for every engagement
- **Documentation-Driven**: Maintain detailed technical notes, write-ups, and cheat sheets for every technique practiced (100+ technical documents)
- **Attacker Mindset**: Trained to think from an adversary's perspective — identifying trust boundaries, broken assumptions, and logic flaws
- **Blue Team Awareness**: Understand detection methods (Windows Event IDs, SIEM correlation, EDR indicators) alongside offensive techniques — enabling comprehensive security analysis
- **Self-Directed Learner**: Built entire cybersecurity skill set through self-driven research, lab work, and structured practice plans without formal training

---

## ADDITIONAL INFORMATION

- **Languages**: English, Hindi, Gujarati
- **Interests**: CTF competitions, bug bounty research, reading disclosed vulnerability reports, building offensive security automation tools
- **GitHub**: Active repository documenting cybersecurity learning journey with 100+ technical write-ups covering penetration testing, Active Directory attacks, web application security, and privilege escalation techniques

---

*References available upon request.*
