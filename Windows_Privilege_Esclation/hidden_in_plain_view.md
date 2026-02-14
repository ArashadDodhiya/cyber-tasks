# 🔎 What Does “Hidden in Plain View” Mean?

It means:

* The vulnerability is already there.
* No exploit or malware is required.
* It’s often just poor configuration.
* A normal domain/user account can discover it.

These are common in real-world environments.

---

# 🚩 Common “Hidden in Plain View” Privilege Escalation Vectors

---

## 1️⃣ Unquoted Service Paths

### 🧠 The Problem

If a Windows service path contains spaces and isn’t quoted:

```
C:\Program Files\My App\service.exe
```

Windows may interpret it as:

```
C:\Program.exe
```

If a low-privileged user can write to `C:\`, they could place a malicious `Program.exe`.

### 🔍 How to Detect (Enumeration)

In PowerShell:

```powershell
wmic service get name,displayname,pathname,startmode
```

Or:

```powersshell
Get-WmiObject win32_service | Select Name, PathName
```

Look for:

* Spaces in the path
* No quotes
* Service running as SYSTEM
* Writable directory in path

### 🛡️ Prevention

* Always quote service paths
* Restrict write permissions
* Monitor service changes

---

## 2️⃣ Weak Service Permissions

If a low-privileged user can:

* Modify a service
* Change its binary path
* Restart it

They can escalate privileges.

### 🔍 Check Permissions

```powershell
sc qc ServiceName
```

To check ACLs:

```powersshell
sc sdshow ServiceName
```

Look for:

* `SERVICE_CHANGE_CONFIG`
* `SERVICE_START`
* `WRITE_DAC`
* `WRITE_OWNER`

If granted to non-admin users → vulnerable.

---

## 3️⃣ Writable Service Binaries

Even if the service path is correct, if users can write to:

```
C:\Program Files\App\service.exe
```

They can replace it.

### 🔍 Check Permissions

```powershell
icacls "C:\Program Files\App\service.exe"
```

Look for:

* `(F)` Full
* `(M)` Modify
* Granted to `Users` or `Authenticated Users`

---

## 4️⃣ AlwaysInstallElevated (Registry Misconfiguration)

If both registry keys are enabled:

```
HKLM\Software\Policies\Microsoft\Windows\Installer
HKCU\Software\Policies\Microsoft\Windows\Installer
```

And `AlwaysInstallElevated = 1`

Any user can install MSI files as SYSTEM.

### 🔍 Check

```powershell
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
reg query HKCU\Software\Policies\Microsoft\Windows\Installer
```

⚠️ This is a classic lab privilege escalation.

---

## 5️⃣ Weak Folder Permissions in Program Files

Developers sometimes:

* Give “Everyone” write access
* Set insecure NTFS permissions

Check:

```powershell
icacls "C:\Program Files"
```

Or recursively:

```powersshell
Get-ChildItem "C:\Program Files" -Recurse | Get-Acl
```

Look for:

* `BUILTIN\Users:(M)`
* `Everyone:(F)`

---

## 6️⃣ Scheduled Tasks with Weak Permissions

If a scheduled task:

* Runs as SYSTEM
* References a writable script

You can modify the script.

### 🔍 Check:

```powershell
schtasks /query /fo LIST /v
```

Look for:

* Task To Run path
* Run As User = SYSTEM
* Writable script path

---

## 7️⃣ Stored Credentials in Files

“Hidden in plain view” often means:

* `C:\Scripts\backup.ps1`
* `web.config`
* `unattend.xml`
* `sysprep.inf`
* `Group Policy Preferences` (old environments)

Look for:

```powershell
findstr /si password *.xml *.ini *.txt
```

This is credential harvesting, not exploitation.

---

# 🔬 How Ethical Hackers Approach This

Instead of randomly attacking, we:

### Step 1: Enumerate

* Services
* Scheduled tasks
* Permissions
* Registry
* Installed software

### Step 2: Identify Misconfigurations

* Write permissions
* Unquoted paths
* Weak ACLs
* Hardcoded credentials

### Step 3: Validate Safely (In Lab)

* Never on production without authorization
* Follow engagement scope
* Document findings

---

# 🛡️ Defensive Perspective (Blue Team)

To prevent these:

* Regular permission audits
* Use least privilege
* Monitor service changes
* Enable Sysmon
* Audit registry changes
* Use tools like:

  * Microsoft Defender
  * Windows Security Baselines
  * CIS Benchmarks

---

# 🧪 Practice Lab Suggestions

Since you're learning privilege escalation, try:

* HackTheBox (Windows machines)
* TryHackMe Windows PrivEsc rooms
* Build a lab:

  * Windows 10
  * Windows Server
  * Domain Controller
  * Standard User account

Practice enumeration only first.

---

# 📌 Key Takeaway

Windows Privilege Escalation is often not about:
❌ Exploits
❌ Zero-days

It’s about:
✔ Misconfigurations
✔ Poor permissions
✔ Forgotten services
✔ Credentials left behind

That’s why it's “hidden in plain view.”