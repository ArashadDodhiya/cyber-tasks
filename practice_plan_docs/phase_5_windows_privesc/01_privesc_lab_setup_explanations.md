# Windows PrivEsc Lab Setup — Explanations & Troubleshooting

This document contains additional explanations and troubleshooting steps for the `01_privesc_lab_setup.md` guide, based on common issues encountered during the setup process.

---

## General Rule of Thumb for Lab Setup

*   **Setup** is done as **Administrator** on the **Windows VM**.
*   **Attacking/Practicing** is done as `trainee` on the **Windows VM**.
*   **Tool Preparation** is done on **Kali**.

### Understanding the Workflow
Whenever a section is marked with the target icon (🎯 **Practice** or **Practice as trainee**), that is the transition point where setup ends and "hacking" begins:

1.  Open `cmd.exe` as **Administrator** on the Windows VM.
2.  Run the setup commands.
3.  Switch user by running `runas /user:trainee cmd.exe` (password: `trainee123`).
4.  **Now you are the "attacker."** Use the `trainee` account to try and exploit the vulnerability to become `SYSTEM` or `Administrator`.
5.  Once you succeed, close that window and go back to the Administrator cmd to set up the next scenario.

---

## Step 2: Weak Service Permissions Explained

### What is a Windows Service?
A Windows service is a program that runs in the background (like a daemon in Linux). Services have properties including:
- **Binary path**: The executable that runs when the service starts
- **Startup type**: Auto, manual, or disabled
- **Permissions (DACL)**: Who can start, stop, or *modify* the service

### The Vulnerability
When a service has weak permissions (e.g., "Everyone" can change its configuration), a low-privilege user can modify the `binpath` to point to a malicious executable. When the service restarts, Windows runs the attacker's payload instead of the original program.

### What `sc sdset` Does
The `sc sdset` command sets the **Security Descriptor** (permissions) of a service using **SDDL** (Security Descriptor Definition Language). The key part in our setup is `(A;;RPWPCCDCLCSWRCWDWOGA;;;WD)`:
- `A` = Allow
- `WD` = **Everyone** (World)
- The permissions include `RPWPCCDCLCSWRCWDWOGA` which gives full control including the ability to change the service configuration

### Troubleshooting: "Access is denied" When Modifying Service
If `trainee` cannot modify the service, verify the DACL was applied:
```cmd
sc sdshow VulnService
```
Look for `WD` (Everyone) with sufficient permissions. If missing, re-run the `sc sdset` command as Administrator.

---

## Step 3: Unquoted Service Paths Explained

### How Windows Resolves Paths
When a service binary path contains spaces and is **not enclosed in quotes**, Windows doesn't know where the executable name ends and where the arguments begin. It tries each possible interpretation from left to right:

For the path: `C:\Program Files\Unquoted Path Service\Common Files\service.exe`

Windows tries (in order):
1. `C:\Program.exe`
2. `C:\Program Files\Unquoted.exe`
3. `C:\Program Files\Unquoted Path.exe`
4. `C:\Program Files\Unquoted Path Service\Common.exe`
5. `C:\Program Files\Unquoted Path Service\Common Files\service.exe`

If you can write to any of those directories, you can place an executable at a path that Windows tries *before* the legitimate one.

### Why Quotes Fix This
If the path were properly quoted:
`"C:\Program Files\Unquoted Path Service\Common Files\service.exe"`

Windows would know the entire string is the path and wouldn't try shorter interpretations.

### Troubleshooting: Service Fails to Start After Exploit
This is expected! Your payload (`Common.exe`) is not a real service binary, so the Service Control Manager may report an error. However, your payload still executes — check your listener on Kali for a reverse shell, or check `net localgroup administrators` for added users.

---

## Step 4: Scheduled Tasks Explained

### Windows Scheduled Tasks vs. Linux Cron Jobs
| Concept         | Linux             | Windows                        |
| --------------- | ----------------- | ------------------------------ |
| Scheduler       | `cron`            | `Task Scheduler`               |
| Config file     | `/etc/crontab`    | `schtasks` / Task Scheduler UI |
| Run as root     | `root` in crontab | `/ru SYSTEM`                   |
| Writable script | `/opt/cleanup.sh` | `C:\Tasks\backup.bat`          |

### The Vulnerability
Same concept as Linux cron exploitation: if a scheduled task runs as SYSTEM and references a script that a low-privilege user can modify, you can inject commands into that script. When the task runs next, Windows executes your commands as SYSTEM.

### Troubleshooting: Task Not Executing
- Check the task status: `schtasks /query /tn "BackupTask" /fo LIST /v`
- Look at "Last Run Time" and "Last Result"
- Make sure the Task Scheduler service is running: `sc query Schedule`
- If the task shows "Disabled", re-enable it: `schtasks /change /tn "BackupTask" /enable`

---

## Step 5: AlwaysInstallElevated Explained

### What is AlwaysInstallElevated?
This is a Windows Group Policy setting that, when enabled, allows **any user** to install MSI (Microsoft Installer) packages with **SYSTEM-level privileges**. It must be enabled in **both** the HKLM (machine) and HKCU (user) registry hives to be exploitable.

### Why Both Registry Keys Are Needed
- `HKLM` key = The machine-level policy says "elevated installs are allowed"
- `HKCU` key = The user-level policy says "this user can use elevated installs"
- If only one is set, the feature is NOT active — **both must be `0x1`**

### Real-World Context
System administrators sometimes enable this to allow users to install approved software without admin credentials. Attackers exploit this by crafting a malicious MSI package that executes arbitrary commands as SYSTEM.

---

## Step 6: Credential Hunting Explained

### Why Credentials Get Left Behind
In real-world environments, credentials end up in many places:
- **Registry**: Applications store connection strings, API keys, or passwords
- **Unattend.xml**: Used during automated Windows installations — often contains the admin password in plaintext or Base64
- **web.config**: IIS web server configuration files may contain database credentials
- **PowerShell history**: Commands typed previously are saved to a history file, and admins sometimes type passwords in commands
- **Credential Manager**: Windows stores credentials for network resources

### The Key Hunting Locations
```
Registry          → reg query HKLM /f password /t REG_SZ /s
Unattend files    → C:\Windows\Panther\Unattend.xml
Web configs       → C:\inetpub\wwwroot\web.config
PS History        → %APPDATA%\...\ConsoleHost_history.txt
Credential Manager→ cmdkey /list
SAM/SYSTEM backups→ C:\Windows\Repair\ or C:\Windows\System32\config\RegBack\
```

### Troubleshooting: `reg query` Returns Too Many Results
The registry search can be slow and return a lot of noise. Tips:
- Search specific hives: `reg query HKLM\SOFTWARE /f password /t REG_SZ /s`
- Pipe to `findstr` for filtering: `reg query HKLM /f password /t REG_SZ /s 2>nul | findstr /i "password pass pwd"`
- Use WinPEAS instead — it checks all common credential locations automatically

---

## Step 7: Token Impersonation Explained

### What is SeImpersonatePrivilege?
In Windows, certain accounts (like IIS service accounts, SQL Server accounts) are given the right to "impersonate" other users. This means they can act on behalf of another user — including SYSTEM.

### Why Service Accounts Have This
Service accounts need to impersonate clients they serve. For example:
- **IIS** needs to impersonate web users to check file permissions
- **SQL Server** needs to impersonate users for Windows Authentication

### How Potato Attacks Work (Simplified)
1. Attacker (with SeImpersonatePrivilege) tricks a SYSTEM-level process into connecting to a fake server
2. The attacker captures the SYSTEM **token** (authentication credential)
3. The attacker uses this token to impersonate SYSTEM and spawn a new process as SYSTEM

### Common Tools
| Tool         | Best For                   | How It Works                           |
| ------------ | -------------------------- | -------------------------------------- |
| PrintSpoofer | Win10 / Server 2016+       | Abuses the Print Spooler service       |
| GodPotato    | All modern Windows         | Universal potato attack, very reliable |
| JuicyPotato  | Win10 < 1809 / Server 2019 | Abuses COM/DCOM                        |
| SweetPotato  | Multiple versions          | Combines multiple potato techniques    |

### Troubleshooting: "SeImpersonatePrivilege" Shows as Disabled
Even if it shows "Disabled" in `whoami /priv`, the privilege is still **assignable** — tools like PrintSpoofer will enable it automatically before using it. If the privilege is completely missing (not listed at all), the attack won't work.

---

## Step 8: Tool Transfer Methods

### Choosing a Transfer Method
| Method      | When to Use                       | Pros                 | Cons                      |
| ----------- | --------------------------------- | -------------------- | ------------------------- |
| HTTP Server | Target has PowerShell or certutil | Simple setup         | May trigger AV            |
| SMB Server  | Target can access network shares  | Very natural on Win  | Requires impacket on Kali |
| evil-winrm  | WinRM is enabled                  | Upload/download cmds | Requires credentials      |
| RDP copy    | RDP is available                  | Drag and drop        | Slow for large files      |

### Troubleshooting: "Connection refused" During File Transfer
1. **HTTP method**: Make sure the Python HTTP server is still running on Kali
2. **SMB method**: Make sure impacket-smbserver is running and use `-smb2support` flag
3. **Firewall**: Verify the Windows firewall is disabled or has the correct rules
4. **Network**: Verify both VMs are on the same `hacklab` internal network

### Troubleshooting: Windows Defender Blocks Tools
Windows Defender may quarantine tools like WinPEAS or PrintSpoofer:
```powershell
# Disable real-time protection (as Administrator)
Set-MpPreference -DisableRealtimeMonitoring $true

# Or add an exclusion for the tools directory
Add-MpPreference -ExclusionPath "C:\temp"
```

> ⚠️ **Remember**: Disabling AV is for lab use only. In real pentests, you'd need to use AV evasion techniques!

---

## Step 9: Remote Access Explained

### RDP vs. WinRM vs. SMB
| Protocol | Port | Tool on Kali          | Best For                          |
| -------- | ---- | --------------------- | --------------------------------- |
| RDP      | 3389 | `xfreerdp`/`rdesktop` | GUI access, file transfer         |
| WinRM    | 5985 | `evil-winrm`          | PowerShell shell, upload/download |
| SMB      | 445  | `smbclient`           | File transfer, share enumeration  |

### Why Disable the Firewall in the Lab?
Windows Firewall blocks incoming connections by default. In a real pentest, you'd find ways around this, but in your lab, turning it off removes unnecessary friction so you can focus on learning PrivEsc techniques.

### Troubleshooting: RDP "Connection refused"
1. Verify RDP is enabled: `reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections`
   - Should show `0x0` (0 = enabled)
2. Check firewall rule: `netsh advfirewall firewall show rule name="RDP"`
3. Make sure Network Level Authentication (NLA) isn't blocking you:
   ```cmd
   reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f
   ```

---

> 🔙 **Back to**: [Windows PrivEsc Lab Setup Guide](./01_privesc_lab_setup.md)
> 🔙 **Back to**: [Windows PrivEsc Main Guide](./windows_privesc.md)
