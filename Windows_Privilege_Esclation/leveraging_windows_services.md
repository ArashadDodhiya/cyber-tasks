# 1️⃣ What Is a Windows Service? (Simple Explanation)

A **Windows service** is a program that:

* Runs in the background
* Often starts automatically
* Usually runs as **SYSTEM** (very powerful account)

Examples:

* Windows Update
* Print Spooler
* Antivirus service
* Database services

Think of a service like a robot worker that runs all the time in the background.

---

# 2️⃣ Why Services Are Important for Privilege Escalation

If a service:

* Runs as SYSTEM
  AND
* Is misconfigured

Then a low-privileged user may be able to:

👉 Modify it
👉 Replace what it runs
👉 Restart it

Result: Your code runs as SYSTEM.

That’s privilege escalation.

---

# 3️⃣ Common Service Misconfigurations (Simple)

Here are the most common ones 👇

---

## 🔴 1. Unquoted Service Path

Service path:

```
C:\Program Files\My App\service.exe
```

If not wrapped in quotes:

```
C:\Program Files\My App\service.exe
```

Windows may try:

```
C:\Program.exe
```

If you can write to `C:\`, you could place `Program.exe` there.

When service starts → your file runs as SYSTEM.

Very common in older systems.

---

## 🔴 2. Writable Service Binary

Service runs:

```
C:\Program Files\App\app.exe
```

If normal users can modify or replace `app.exe`:

You replace it with your malicious version.

Restart service → SYSTEM shell.

---

## 🔴 3. Writable Service Folder

Even if the file isn’t writable:

If the folder is writable, you can:

* Delete original file
* Replace it

Same result.

---

## 🔴 4. Weak Service Permissions

Sometimes users are allowed to:

* Change service configuration
* Restart service
* Modify binary path

If you can change the binary path to:

```
C:\Users\Public\myshell.exe
```

Then restart service → SYSTEM execution.

---

# 4️⃣ How to Check for Vulnerable Services

## 🔍 List Services

```cmd
sc query
```

---

## 🔍 Get Detailed Info

```cmd
sc qc ServiceName
```

Shows:

* Binary path
* Start type
* Account it runs as

---

## 🔍 Check Permissions on Service File

```cmd
icacls "C:\Path\To\Service.exe"
```

Look for:

* Users:(F)
* Users:(M)
* Everyone:(F)

That’s bad.

---

## 🔍 Check Service Permissions

Using PowerUp (lab tool):

```powershell
Invoke-AllChecks
```

Or:

```powershell
Get-Service
```

---

# 5️⃣ Simple Exploitation Example (Lab Only ⚠️)

Scenario:

Service runs as SYSTEM
Binary:

```
C:\Program Files\VulnApp\vuln.exe
```

File is writable.

Steps:

1. Backup original file
2. Replace with malicious EXE
3. Restart service:

```cmd
net stop VulnService
net start VulnService
```

If vulnerable → your code runs as SYSTEM.

---

# 6️⃣ Why This Works (In One Sentence)

Because Windows trusts the service configuration.

If you can change what the service runs,
you control what SYSTEM runs.

---

# 7️⃣ Defensive View (Very Important)

To prevent this:

✅ Always quote service paths
✅ Restrict write permissions on Program Files
✅ Restrict who can modify services
✅ Monitor service changes
✅ Use least privilege accounts

---

# 8️⃣ Quick Comparison of Service Attacks

| Type                      | Difficulty | Common?     |
| ------------------------- | ---------- | ----------- |
| Unquoted Path             | Easy       | Very common |
| Writable Binary           | Easy       | Common      |
| Weak Permissions          | Medium     | Common      |
| DLL Hijacking via Service | Advanced   | Common      |

---

# 9️⃣ Real-World Tip

In modern Windows:

* Program Files is usually protected
* Defender detects many service modifications

Most real findings are in:

* Third-party software
* Legacy apps
* Misconfigured enterprise environments
