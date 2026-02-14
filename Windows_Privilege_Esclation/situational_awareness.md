# 🔎 What is Situational Awareness?

In privilege escalation, **situational awareness** means:

> Understanding the current system, user privileges, misconfigurations, and attack surface before attempting escalation.

You are essentially asking:

* Who am I?
* What can I access?
* What is misconfigured?
* What can I abuse legally in this environment?

---

# 🧠 Step 1: Identify Current Context

### 🔹 Who am I?

```cmd
whoami
whoami /groups
whoami /priv
```

Look for:

* SeImpersonatePrivilege
* SeDebugPrivilege
* SeBackupPrivilege
* SeRestorePrivilege
* SeTakeOwnershipPrivilege

⚠️ These privileges often lead to escalation opportunities.

---

# 🖥 Step 2: System Information

```cmd
systeminfo
hostname
ver
```

Check:

* Windows version & build number
* Patch level
* Domain membership

Why?
Older builds may be vulnerable to known exploits (in lab settings only).

---

# 👥 Step 3: User & Group Enumeration

```cmd
net user
net localgroup
net localgroup administrators
```

PowerShell:

```powershell
Get-LocalUser
Get-LocalGroup
```

Questions to ask:

* Am I already in a privileged group?
* Are there other users logged in?
* Is there a service account I can target?

---

# 🔐 Step 4: Check Running Processes

```cmd
tasklist /v
```

PowerShell:

```powershell
Get-Process
```

Look for:

* High-privilege processes
* Antivirus / EDR presence
* Custom services

Understanding defenses is part of situational awareness.

---

# 📂 Step 5: Services Enumeration

```cmd
sc query
wmic service get name,displayname,pathname,startmode
```

Look for:

* Unquoted service paths
* Writable service binaries
* Services running as SYSTEM

These are classic escalation vectors.

---

# 🛠 Step 6: Installed Programs

```cmd
wmic product get name,version
```

Or check:

```
C:\Program Files
C:\Program Files (x86)
```

Look for:

* Vulnerable software versions
* Backup agents
* Misconfigured third-party tools

---

# 📁 Step 7: File & Folder Permissions

Check interesting directories:

```cmd
icacls "C:\Program Files\SomeService"
```

Look for:

* Modify / Full control permissions for low-priv users
* Writable directories in PATH

---

# 🔑 Step 8: Credential Hunting

⚠️ Only in authorized labs.

Search for:

* Unattend.xml
* Web.config
* .rdp files
* Saved scripts
* Registry saved credentials

```cmd
reg query HKLM /f password /t REG_SZ /s
```

---

# 🌐 Step 9: Network Awareness

```cmd
ipconfig /all
netstat -ano
route print
arp -a
```

Understand:

* Is it domain joined?
* Is it multi-homed?
* Are there internal services?

---

# 🧰 Automated Tools (Lab Use Only)

For practice environments:

* **WinPEAS**
* **Seatbelt**
* **PowerUp**
* **SharpUp**

These automate situational awareness and highlight:

* Weak permissions
* Credential leaks
* Privilege abuse vectors

💡 Always review the output manually — tools miss context.

---

# 🎯 Key Privilege Escalation Vectors to Look For

| Vector                | What to Look For                 |
| --------------------- | -------------------------------- |
| Token Impersonation   | SeImpersonatePrivilege           |
| Services              | Unquoted paths, weak ACLs        |
| Scheduled Tasks       | Writable task binaries           |
| DLL Hijacking         | Writable directories in PATH     |
| Registry              | Weak permissions                 |
| AlwaysInstallElevated | Registry misconfig               |
| UAC Bypass            | Admin context but filtered token |
| Credentials           | Hardcoded passwords              |

---

# 🛡 Defensive Mindset (Blue Team Awareness)

Understanding situational awareness also helps defenders:

* Remove unnecessary privileges
* Harden service permissions
* Monitor abnormal enumeration behavior
* Enable PowerShell logging
* Audit token privileges

---

# 🔥 Practical Lab Recommendation

If you're learning:

* HackTheBox
* TryHackMe (Windows PrivEsc rooms)
* VulnHub
* Build your own Windows AD lab in VirtualBox

---

# 🧠 Pro Tip

Privilege escalation is not about running exploits immediately.

It’s about:

1. Enumerate
2. Analyze
3. Identify misconfiguration
4. Choose the safest vector
5. Escalate cleanly

