# 🧠 What is Situational Awareness in PrivEsc?

Situational awareness means:

> Understanding where you are, who you are, what you can access, and what opportunities exist.

Before escalating privileges, you gather information.

Think:

* Who am I?
* What files can I read?
* What users exist?
* What credentials might be exposed?

Now let’s break down each command.

---

# 1️⃣ Get-ChildItem (PowerShell) – Finding Specific File Extensions

`Get-ChildItem` (alias: `gci`, `dir`, `ls`) is used to list files and directories.

It’s extremely powerful for enumeration.

---

## 🔎 Find Files with a Specific Extension

### Find all `.txt` files in current directory

```powershell
Get-ChildItem -Filter *.txt
```

---

### 🔍 Search Recursively (Very Important in PrivEsc)

```powershell
Get-ChildItem -Path C:\ -Filter *.xml -Recurse -ErrorAction SilentlyContinue
```

Explanation:

* `-Path C:\` → Start from C drive
* `-Filter *.xml` → Only XML files
* `-Recurse` → Search subfolders
* `-ErrorAction SilentlyContinue` → Hide access denied errors

---

## 🔥 Common Extensions to Search in PrivEsc

```powershell
*.xml
*.ini
*.config
*.txt
*.ps1
*.bat
*.vbs
*.cred
*.kdbx
```

Why?

Because they may contain:

* Hardcoded passwords
* Service credentials
* Backup scripts
* Scheduled task scripts

---

## 🎯 Example: Search for Passwords in Files

```powershell
Get-ChildItem -Path C:\ -Recurse -Include *.xml,*.ini,*.txt -ErrorAction SilentlyContinue
```

Then inspect interesting files.

---

# 2️⃣ `type` Command (CMD)

The `type` command displays file contents.

### Basic Usage

```cmd
type filename.txt
```

Example:

```cmd
type backup.ps1
```

This helps you:

* Read scripts
* Check configuration files
* Find stored credentials
* Identify service accounts

---

## 🧠 Why Important in PrivEsc?

Example scenario:

You find:

```
C:\Scripts\backup.ps1
```

You run:

```cmd
type backup.ps1
```

And see:

```powershell
$pass = "Admin@123"
```

Now you discovered credentials.

This is passive enumeration — no exploitation.

---

# 3️⃣ `net user` Commands

Very important for situational awareness.

---

## 🔎 Who Am I?

```cmd
whoami
```

---

## 🔎 List All Users

```cmd
net user
```

This shows:

* Local accounts
* Potential admin accounts
* Service accounts

---

## 🔎 Get Details About a Specific User

```cmd
net user username
```

Example:

```cmd
net user administrator
```

Look for:

* Local Group Memberships
* Password last set
* Account active

---

## 🔎 Check Local Administrators

```cmd
net localgroup administrators
```

This is crucial.

It tells you:

* Who has admin rights?
* Are there misconfigured users?

---

# 🧠 How These Commands Fit into PrivEsc

Imagine you're a low-privileged user.

You would:

### Step 1 – Identify Yourself

```cmd
whoami
whoami /priv
whoami /groups
```

---

### Step 2 – Enumerate Users

```cmd
net user
net localgroup administrators
```

---

### Step 3 – Search for Credentials

```powershell
Get-ChildItem -Recurse -Include *.xml,*.ini,*.txt
```

Then:

```cmd
type filename.txt
```

---

# ⚠️ Ethical Reminder

These commands are:

* Legitimate system commands
* Used by administrators daily
* Also used in penetration testing

Never use them on systems without authorization.

---

# 🔍 Real-World Example Scenario

You log in as:

```
User: john
Group: Users
```

You run:

```cmd
net localgroup administrators
```

You see:

```
BackupAdmin
```

Then:

```powershell
Get-ChildItem -Path C:\ -Recurse -Filter *.ps1
```

You find:

```
C:\Backup\backup.ps1
```

Then:

```cmd
type C:\Backup\backup.ps1
```

You discover stored credentials.

This is classic "hidden in plain view."

---

# 📌 Key Takeaway

| Command        | Purpose in PrivEsc            |
| -------------- | ----------------------------- |
| Get-ChildItem  | Find interesting files        |
| type           | Read file contents            |
| net user       | Enumerate users               |
| net localgroup | Check admin access            |
| whoami         | Identify your privilege level |
