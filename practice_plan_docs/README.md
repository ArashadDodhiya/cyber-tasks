# 🎯 Comprehensive Cybersecurity Practice Plan

> **A structured, hands-on practice plan covering 8 phases of offensive security.**
> Each phase has its own dedicated folder with detailed exercises, tool usage, and lab references.

---

## 🗺️ Learning Path Overview

```
Phase 1: Information Gathering (Week 1-2)        → Recon, scanning, enumeration
    ↓
Phase 2: Password Attacks (Week 3)               → Cracking, brute forcing, spraying
    ↓
Phase 3: File Transfer Techniques (Week 4)        → Moving files between attacker & target
    ↓
Phase 4: Linux Privilege Escalation (Week 5-6)    → Low-priv → root on Linux
    ↓
Phase 5: Windows Privilege Escalation (Week 7-8)  → Low-priv → SYSTEM on Windows
    ↓
Phase 6: Active Directory Attacks (Week 9-12)     → Domain enumeration → Domain Admin
    ↓
Phase 7: Port Redirection & Tunneling (Week 13)   → Pivoting & tunneling through networks
    ↓
Phase 8: Full Attack Chains (Week 14+)            → End-to-end penetration testing
```

---

## 🧪 Lab Environment

### What You Need

| Component            | Purpose                                | Status  |
| -------------------- | -------------------------------------- | ------- |
| **Kali Linux**       | Attacker machine                       | ✅ Setup |
| **Metasploitable 2** | Vulnerable Linux target                | ✅ Setup |
| **DVWA**             | Vulnerable web app (on Metasploitable) | ✅ Setup |

### Free Vulnerable Platforms Used in This Plan

| Platform                | Type                     | URL                                             | Used In          |
| ----------------------- | ------------------------ | ----------------------------------------------- | ---------------- |
| **DVWA**                | Self-hosted web app      | Pre-installed on Metasploitable                 | Phase 1, 2, 3, 8 |
| **OWASP Juice Shop**    | Self-hosted web app      | `docker run -p 3000:3000 bkimminich/juice-shop` | Phase 1, 2, 8    |
| **bWAPP**               | Self-hosted web app      | `docker run -p 80:80 raesene/bwapp`             | Phase 1, 2, 8    |
| **WebGoat**             | Self-hosted web app      | `docker run -p 8080:8080 webgoat/webgoat`       | Phase 2, 8       |
| **Mutillidae II**       | Self-hosted web app      | `docker run -p 80:80 citizenstig/nowasp`        | Phase 1, 2, 8    |
| **PortSwigger Academy** | Online labs (free)       | https://portswigger.net/web-security            | Phase 1, 2, 8    |
| **TryHackMe**           | Online labs (free rooms) | https://tryhackme.com                           | Phase 1–8        |
| **Hack The Box**        | Online labs (free tier)  | https://hackthebox.com                          | Phase 4–8        |
| **VulnHub**             | Downloadable VMs         | https://vulnhub.com                             | Phase 4, 5, 8    |
| **OverTheWire**         | Online wargames          | https://overthewire.org                         | Phase 1, 4       |
| **PicoCTF**             | Online CTF               | https://picoctf.org                             | Phase 2, 8       |
| **HackThisSite**        | Online challenges        | https://hackthissite.org                        | Phase 1, 2       |
| **Exploit Education**   | Downloadable VMs         | https://exploit.education                       | Phase 4, 5       |
| **CyberDefenders**      | Online DFIR challenges   | https://cyberdefenders.org                      | Phase 8          |
| **DVWA (Docker)**       | Alternative DVWA setup   | `docker run -p 80:80 vulnerables/web-dvwa`      | Phase 1, 2, 8    |
| **Metasploitable 3**    | Advanced vuln VM         | https://github.com/rapid7/metasploitable3       | Phase 5, 6       |
| **Google Gruyere**      | Online web app           | https://google-gruyere.appspot.com              | Phase 1, 2       |
| **XSS Game**            | Online XSS practice      | https://xss-game.appspot.com                    | Phase 2          |
| **CMD Challenge**       | Online Linux practice    | https://cmdchallenge.com                        | Phase 1          |

---

## 📂 Folder Structure

```
practice_plan_docs/
├── README.md                          ← You are here
├── phase_1_information_gathering/
│   └── information_gathering.md       ← Passive & active recon, scanning, enumeration
├── phase_2_password_attacks/
│   └── password_attacks.md            ← Hash cracking, brute forcing, spraying
├── phase_3_file_transfer/
│   └── file_transfer.md               ← All transfer methods between machines
├── phase_4_linux_privesc/
│   └── linux_privesc.md               ← Linux privilege escalation techniques
├── phase_5_windows_privesc/
│   └── windows_privesc.md             ← Windows privilege escalation techniques
├── phase_6_active_directory/
│   └── active_directory.md            ← AD enumeration, attacks, lateral movement
├── phase_7_tunneling/
│   └── tunneling.md                   ← Port forwarding, pivoting, SOCKS proxies
└── phase_8_attack_chains/
    └── attack_chains.md               ← Full end-to-end penetration tests
```

---

## 📝 How to Use This Plan

1. **Go phase by phase** — don't skip ahead
2. **Set up the vulnerable targets** listed in each phase before starting
3. **Follow the exercises in order** — they build on each other
4. **Document everything** — create writeups for each exercise
5. **Track your progress** — check off exercises as you complete them
6. **Repeat difficult exercises** — repetition builds muscle memory

---

## ⚠️ Legal Disclaimer

> **Only practice on systems you own or have explicit written permission to test.**
> All exercises in this plan use intentionally vulnerable, isolated lab environments.
> Unauthorized access to computer systems is illegal.

---

## 🔗 Cross-References to Your Existing Notes

| Topic                 | Your Notes Folder                   |
| --------------------- | ----------------------------------- |
| Information Gathering | `information_gathering/`            |
| Password Attacks      | `attacks/`                          |
| File Transfer         | `file_transfer/`                    |
| Linux PrivEsc         | `linux_prev_esca/`                  |
| Windows PrivEsc       | `Windows_Privilege_Esclation/`      |
| Active Directory      | `active_directory/` + `AD_hacking/` |
| Port Redirection      | `port_redir_tunneling/`             |
| Home Lab Setup        | `home_lab/`                         |
