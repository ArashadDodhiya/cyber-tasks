# 🧠 Why PowerShell is So Powerful

PowerShell gives you:

* Full access to Windows APIs
* Object-based output (not just text)
* Deep system visibility
* Registry + service inspection
* File permission auditing

It turns Windows into an open book.

---

# 🎯 Step 1 – Situational Awareness (Who Am I?)

### Identify current user

```powershell
whoami
```

### Token privileges

```powershell
whoami /priv
```

Look for interesting privileges like:

* SeImpersonatePrivilege
* SeBackupPrivilege
* SeRestorePrivilege
* SeShutdownPrivilege

---

# 👥 Step 2 – User & Group Enumeration

### List local users

```powershell
Get-LocalUser
```

### List local admins

```powershell
Get-LocalGroupMember Administrators
```

### Domain info (if domain joined)

```powershell
Get-ADUser -Filter *
```

(If RSAT is installed)

---

# 🔥 Step 3 – Service Gold Mine

Services are one of the biggest PrivEsc vectors.

---

## 🔎 List All Services with Paths

```powershell
Get-WmiObject win32_service | Select Name, StartName, PathName
```

Look for:

* Running as LocalSystem
* Unquoted paths
* Custom directories
* Third-party software

---

## 🔎 Find Unquoted Service Paths

```powershell
Get-WmiObject win32_service | Where-Object {
    $_.PathName -notmatch '"' -and $_.PathName -match ' '
} | Select Name, PathName
```

---

## 🔎 Check Writable Service Binaries

```powershell
Get-WmiObject win32_service | ForEach-Object {
    $path = $_.PathName.Split('"')[1]
    Get-Acl $path
}
```

Look for:

* Users with Modify or FullControl

---

# 📂 Step 4 – File System Gold Mine

---

## 🔍 Find Interesting Files

```powershell
Get-ChildItem -Path C:\ -Recurse -Include *.xml,*.ini,*.config,*.txt -ErrorAction SilentlyContinue
```

---

## 🔍 Search for Password Strings

```powershell
Select-String -Path C:\*.txt -Pattern "password" -Recurse
```

---

## 🔍 Check Writable Directories in Program Files

```powershell
Get-ChildItem "C:\Program Files" | Get-Acl
```

---

# 🗓️ Step 5 – Scheduled Tasks

Scheduled tasks often run as SYSTEM.

---

## List Tasks

```powershell
Get-ScheduledTask
```

---

## Detailed Info

```powershell
Get-ScheduledTask | Select TaskName, TaskPath, State
```

Check:

* What binary/script runs?
* Who runs it?
* Is script writable?

---

# 🧾 Step 6 – Registry Gold Mine

Registry contains:

* Autoruns
* Service configs
* Installer misconfigs
* Stored creds

---

## Check AlwaysInstallElevated

```powershell
Get-ItemProperty HKLM:\Software\Policies\Microsoft\Windows\Installer
Get-ItemProperty HKCU:\Software\Policies\Microsoft\Windows\Installer
```

If both = 1 → misconfiguration.

---

## Startup Locations

```powershell
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

---

# 🔑 Step 7 – Credential Gold Mine

---

## Search for Stored Credentials

```powershell
cmdkey /list
```

---

## PowerShell History

```powershell
Get-Content (Get-PSReadlineOption).HistorySavePath
```

Admins often type passwords in clear text.

This is HUGE in real environments.

---

# 🧠 Step 8 – Token Privilege Opportunities

Check:

```powershell
whoami /priv
```

If you see:

* SeImpersonatePrivilege
* SeAssignPrimaryTokenPrivilege

These can be abused (in real pentests with tools), but for OSCP you mainly need to **recognize them**.

---

# 📊 Why PowerShell is a Gold Mine in OSCP

Because it allows you to:

| Area     | Why Important       |
| -------- | ------------------- |
| Services | Common misconfigs   |
| Tasks    | SYSTEM execution    |
| ACLs     | Writable paths      |
| Registry | Installer abuse     |
| Files    | Stored credentials  |
| Tokens   | Advanced escalation |

All without uploading tools.

---

# 🧩 OSCP Mindset

If you’re stuck:

Run these 5 PowerShell checks immediately:

```powershell
whoami /priv
Get-LocalGroupMember Administrators
Get-WmiObject win32_service
Get-ScheduledTask
Get-ChildItem C:\ -Recurse -Include *.config,*.xml -ErrorAction SilentlyContinue
```

You’ll almost always find something.

---

# 🛡️ Defensive Insight

Blue teams should:

* Audit service permissions
* Restrict folder ACLs
* Remove plaintext credentials
* Monitor PowerShell logging
* Enable Script Block Logging
* Monitor unusual service modifications