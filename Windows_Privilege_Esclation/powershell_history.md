# 🧠 1️⃣ What is PowerShell History?

PowerShell keeps a record of previously executed commands.

There are **two types** of history:

| Type                    | Scope                     | Persistent? |
| ----------------------- | ------------------------- | ----------- |
| Session History         | Current PowerShell window | ❌ No        |
| PSReadLine History File | Stored on disk            | ✅ Yes       |

The **PSReadLine history file** is what attackers love.

---

# 📍 2️⃣ Session History (Current Window Only)

### View current session history

```powershell
Get-History
```

Alias:

```powershell
h
```

Shows:

* Command ID
* Full command text

---

### Re-run a previous command

```powershell
Invoke-History 5
```

Or:

```powershell
r 5
```

⚠️ This only works in the same session.

Once the window closes → session history is gone.

---

# 💾 3️⃣ Persistent PowerShell History (The Real Gold Mine)

PowerShell (v5+) uses **PSReadLine** module.

It saves command history to disk automatically.

---

## 🔎 Find the History File Location

```powershell
(Get-PSReadlineOption).HistorySavePath
```

Typical location:

```
C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

This file persists across sessions.

---

# 📖 4️⃣ Read PowerShell History

### Basic read:

```powershell
Get-Content (Get-PSReadlineOption).HistorySavePath
```

Or:

```powershell
type (Get-PSReadlineOption).HistorySavePath
```

---

# 🔍 5️⃣ Search for Passwords in History

This is extremely common in real environments.

### Search for password strings

```powershell
Select-String -Path (Get-PSReadlineOption).HistorySavePath -Pattern "pass"
```

### Search for credential usage

```powershell
Select-String -Path (Get-PSReadlineOption).HistorySavePath -Pattern "SecureString"
```

### Search for net use commands

```powershell
Select-String -Path (Get-PSReadlineOption).HistorySavePath -Pattern "net use"
```

---

# 🔥 6️⃣ Why It’s So Dangerous

Admins commonly run:

```powershell
net use \\server\share /user:backupadmin P@ssw0rd123
```

Or:

```powershell
$pass = ConvertTo-SecureString "Admin@2024" -AsPlainText -Force
```

Or:

```powershell
Invoke-Sqlcmd -Username sa -Password SuperSecret!
```

All of this gets stored in:

```
ConsoleHost_history.txt
```

Even after logout.

---

# 👥 7️⃣ Check Other Users' History (If Accessible)

If you have access to other user folders:

```powershell
Get-ChildItem C:\Users\
```

Then manually check:

```powersshell
type C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

If readable → jackpot.

---

# 🧠 8️⃣ Check If History Is Enabled

```powershell
Get-PSReadlineOption
```

Important fields:

* HistorySavePath
* HistorySaveStyle
* MaximumHistoryCount

---

# 🔧 9️⃣ History Configuration Options

### Disable history (defensive)

```powershell
Set-PSReadlineOption -HistorySaveStyle SaveNothing
```

### Change max history size

```powersshell
Set-PSReadlineOption -MaximumHistoryCount 1000
```

---

# 🧩 10️⃣ OSCP-Style Scenario

You log in as:

```
User: support
```

You run:

```powershell
Get-Content (Get-PSReadlineOption).HistorySavePath
```

You find:

```powershell
net use \\dc01\backups /user:corp\backupadmin Backup@123
```

You check:

```cmd
net localgroup administrators
```

You see:

```
corp\backupadmin
```

That’s escalation without exploiting anything.

Pure enumeration.

---

# 🧠 11️⃣ Important Concepts to Understand

### 🔹 PowerShell History is NOT encrypted

It is plain text.

### 🔹 Each user has their own history file

Per profile.

### 🔹 It only logs interactive commands

Not background scripts.

### 🔹 Works on PowerShell 5+

Older systems may not use PSReadLine.

---

# 🔍 12️⃣ Advanced Enumeration Trick

Check if PowerShell logging is enabled (blue team view):

```powershell
Get-ItemProperty HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
```

Admins may log everything — including attacker commands.

---

# 🛡️ Defensive Recommendations

Blue teams should:

* Disable history saving on admin accounts
* Use credential vaults instead of typing passwords
* Restrict access to user profile folders
* Enable PowerShell logging
* Monitor access to ConsoleHost_history.txt

---

# 🎯 Exam Strategy (Very Important)

On OSCP-style machines, ALWAYS check:

```powershell
(Get-PSReadlineOption).HistorySavePath
type (Get-PSReadlineOption).HistorySavePath
```

It takes 5 seconds.

It has solved many real exams.

---

# 📌 Quick Command Summary

| Purpose              | Command                                  |
| -------------------- | ---------------------------------------- |
| View session history | `Get-History`                            |
| Find history file    | `(Get-PSReadlineOption).HistorySavePath` |
| Read history         | `Get-Content <path>`                     |
| Search for passwords | `Select-String -Pattern "pass"`          |
| Check PS settings    | `Get-PSReadlineOption`                   |
| Check other users    | Browse `C:\Users\`                       |

---

# 🧠 Key Takeaway

PowerShell history is:

✔ Silent
✔ Persistent
✔ Plain text
✔ Frequently ignored
✔ Extremely valuable
