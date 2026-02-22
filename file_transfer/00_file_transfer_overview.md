# 📂 File Transfer in Ethical Hacking — Complete Beginner's Guide

## 📖 What Is File Transfer in Hacking?

During a penetration test, you almost always need to **move files between machines**:

> **File Transfer** = Getting tools, scripts, or data from one computer to another — usually from your attacker machine to a target (or back).

---

## 🧠 Why Do We Need File Transfer?

```text
┌──────────────┐                              ┌──────────────┐
│   ATTACKER   │  ──── Transfer tools ────▶   │   TARGET     │
│   (Kali)     │                              │   (Victim)   │
│              │  ◀── Exfiltrate data ─────   │              │
│  Has tools:  │                              │  Has data:   │
│  Mimikatz    │                              │  Passwords   │
│  LinPEAS     │                              │  Documents   │
│  WinPEAS     │                              │  Database    │
│  Exploits    │                              │  SAM/NTDS    │
└──────────────┘                              └──────────────┘
```

### Common Reasons

| Reason | Direction | Example |
| --- | --- | --- |
| Upload exploit | Attacker → Target | Send privilege escalation tool |
| Upload enumeration tool | Attacker → Target | Send LinPEAS/WinPEAS |
| Download loot | Target → Attacker | Grab password databases |
| Upload reverse shell | Attacker → Target | Send payload for callback |
| Download config files | Target → Attacker | Grab web.config, .env files |
| Upload persistence tools | Attacker → Target | Send backdoor/implant |

---

## 🗺 File Transfer Methods Overview

```text
┌─────────────────────────────────────────────────────────────┐
│              FILE TRANSFER METHODS                          │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  📡 NETWORK-BASED (Most Common)                      │   │
│  │  • HTTP/HTTPS (Python server, Apache, Nginx)         │   │
│  │  • SMB (Windows file sharing)                        │   │
│  │  • FTP (File Transfer Protocol)                      │   │
│  │  • SCP/SFTP (Secure copy over SSH)                   │   │
│  │  • Netcat (raw TCP transfer)                         │   │
│  │  • TFTP (Trivial FTP — no auth)                      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  💻 BUILT-IN OS TOOLS (No extra install needed)      │   │
│  │  • PowerShell (Windows)                              │   │
│  │  • Certutil (Windows)                                │   │
│  │  • Curl / Wget (Linux)                               │   │
│  │  • Bitsadmin (Windows)                               │   │
│  │  • Copy / Xcopy (Windows)                            │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  🔐 ENCODED/STEALTH METHODS (Bypass detection)       │   │
│  │  • Base64 encoding                                   │   │
│  │  • DNS exfiltration                                  │   │
│  │  • ICMP tunneling                                    │   │
│  │  • Steganography                                     │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  🌐 CLOUD/WEB-BASED                                  │   │
│  │  • Upload to Pastebin / GitHub Gist                  │   │
│  │  • Cloud storage (when nothing else works)           │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 Which Method to Use? (Decision Diagram)

```text
START: Need to transfer a file
  │
  ├── Target is WINDOWS?
  │     │
  │     ├── Has PowerShell? ──▶ Use PowerShell download  ⭐ EASIEST
  │     ├── Has certutil?   ──▶ Use certutil             ⭐ RELIABLE
  │     ├── Has SMB access? ──▶ Use SMB share            ⭐ FAST
  │     ├── Has outbound HTTP? ▶ Use HTTP server + wget   
  │     └── Nothing works?  ──▶ Use Base64 copy-paste    
  │
  ├── Target is LINUX?
  │     │
  │     ├── Has wget?       ──▶ Use wget + HTTP server   ⭐ EASIEST
  │     ├── Has curl?       ──▶ Use curl + HTTP server   ⭐ EASIEST
  │     ├── Has SSH?        ──▶ Use SCP                  ⭐ SECURE
  │     ├── Has netcat?     ──▶ Use netcat transfer      
  │     └── Nothing works?  ──▶ Use Base64 copy-paste    
  │
  └── Need STEALTH?
        │
        ├── Use DNS exfiltration
        ├── Use ICMP tunneling
        └── Use Base64 + encoded channels
```

---

## 📚 Documents in This Folder

| # | Document | What You'll Learn | Level |
| --- | --- | --- | --- |
| 1 | `01_http_file_transfer.md` | Python HTTP server, wget, curl | ⭐ Beginner |
| 2 | `02_windows_file_transfer.md` | PowerShell, certutil, bitsadmin | ⭐ Beginner |
| 3 | `03_linux_file_transfer.md` | wget, curl, SCP, netcat | ⭐ Beginner |
| 4 | `04_SMB_file_transfer.md` | SMB shares, impacket-smbserver | ⭐⭐ Intermediate |
| 5 | `05_FTP_TFTP_transfer.md` | FTP server, TFTP transfers | ⭐⭐ Intermediate |
| 6 | `06_netcat_file_transfer.md` | Netcat/Socat raw transfers | ⭐⭐ Intermediate |
| 7 | `07_SSH_SCP_SFTP_transfer.md` | SCP, SFTP, SSH-based transfers | ⭐⭐ Intermediate |
| 8 | `08_encoding_stealth_transfer.md` | Base64, hex, steganography | ⭐⭐⭐ Advanced |
| 9 | `09_exfiltration_techniques.md` | DNS, ICMP, cloud exfil | ⭐⭐⭐ Advanced |
| 10 | `10_file_transfer_cheatsheet.md` | Quick reference for all methods | All Levels |

---

## 🎓 Suggested Learning Order

```text
BEGINNER:
  1. Read this overview (you're here!)
  2. HTTP transfers     → 01_http_file_transfer.md
  3. Windows methods    → 02_windows_file_transfer.md
  4. Linux methods      → 03_linux_file_transfer.md

INTERMEDIATE:
  5. SMB transfers      → 04_SMB_file_transfer.md
  6. FTP/TFTP           → 05_FTP_TFTP_transfer.md
  7. Netcat transfers   → 06_netcat_file_transfer.md
  8. SSH/SCP/SFTP       → 07_SSH_SCP_SFTP_transfer.md

ADVANCED:
  9. Encoding/Stealth   → 08_encoding_stealth_transfer.md
  10. Exfiltration       → 09_exfiltration_techniques.md

REFERENCE:
  11. Cheatsheet         → 10_file_transfer_cheatsheet.md
```

---

## ⚠ Ethical Reminder

* ✅ Only transfer files on systems you have **written authorization** to test
* ✅ Practice in your own lab environment
* ❌ Never exfiltrate real data without permission
* 🎓 Understanding file transfer helps both red team and blue team
