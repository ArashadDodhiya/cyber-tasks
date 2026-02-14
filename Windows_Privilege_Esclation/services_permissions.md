# 1️⃣ What Are Service Permissions?

When we talk about service permissions, there are **two completely different things**:

### 🔹 A) File System Permissions

Who can modify the service executable file?

Example:

```
C:\Program Files\App\service.exe
```

Controlled by:

```
NTFS permissions (icacls)
```

---

### 🔹 B) Service Object Permissions (More Advanced)

Who can control the service itself?

For example:

* Start it
* Stop it
* Change its configuration
* Change what binary it runs
* Delete it

These are stored in the **Service Control Manager (SCM)**.

👉 Most beginners only check file permissions.
👉 Advanced privilege escalation often involves service object permissions.

---

# 2️⃣ Understanding Service Object Permissions

Each service has a **Security Descriptor**.

It defines which users/groups can:

* Start the service
* Stop the service
* Change config
* Delete service
* Modify permissions

Think of it like ACLs, but for services instead of files.

---

# 3️⃣ Important Service Permissions (What Matters for PrivEsc)

Here are the dangerous ones:

| Permission            | Why It’s Dangerous             |
| --------------------- | ------------------------------ |
| SERVICE_CHANGE_CONFIG | You can change the binary path |
| SERVICE_START         | You can start it               |
| SERVICE_STOP          | You can stop it                |
| SERVICE_ALL_ACCESS    | Full control                   |
| WRITE_DAC             | Modify permissions             |
| WRITE_OWNER           | Take ownership                 |

The most powerful for escalation:

🔥 `SERVICE_CHANGE_CONFIG`

If you can change:

```
Binary Path Name
```

You can point it to your malicious executable.

---

# 4️⃣ How to Check Service Permissions

## 🔍 Method 1: Using sc

```cmd
sc sdshow ServiceName
```

Example output:

```
D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)
  (A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)
  (A;;CCLCSWLOCRRC;;;AU)
```

This is SDDL format (Security Descriptor Definition Language).

Looks scary 😅 but we decode it.

---

### Common Abbreviations:

| Code | Meaning                 |
| ---- | ----------------------- |
| SY   | SYSTEM                  |
| BA   | Built-in Administrators |
| AU   | Authenticated Users     |
| WD   | Everyone                |

Permissions inside:

| Code | Meaning       |
| ---- | ------------- |
| RP   | Start service |
| WP   | Stop service  |
| CC   | Change config |
| DC   | Delete        |
| WD   | Write DAC     |

If you see:

```
(A;;CC;;;AU)
```

That means:
Authenticated Users can change config.

🚨 That’s vulnerable.

---

## 🔍 Method 2: PowerUp (Easier)

In a lab:

```powershell
Invoke-AllChecks
```

It will clearly tell you:

> Service has weak permissions

Much easier than reading SDDL manually.

---

# 5️⃣ Real Privilege Escalation Logic (LAB ONLY ⚠️)

Scenario:

Service runs as:

```
LocalSystem
```

You check permissions:

```
sc sdshow VulnService
```

You discover:
Authenticated Users have `SERVICE_CHANGE_CONFIG`.

Now you can:

Change binary path:

```cmd
sc config VulnService binPath= "C:\Users\Public\shell.exe"
```

Then start it:

```cmd
net start VulnService
```

If successful → your executable runs as SYSTEM.

That’s full privilege escalation.

---

# 6️⃣ Important: Service Start Permissions

Even if you can change config…

If you cannot:

* Start the service
* Or reboot machine

You may not be able to trigger it.

So always check:

* Can I start it?
* Is it auto-start?
* Can I stop it?

---

# 7️⃣ Less Obvious Dangerous Case

Sometimes users cannot change config…

BUT they can:

* Stop the service
* Replace executable (if file writable)
* Start service again

That combination is enough.

---

# 8️⃣ Defensive View (Blue Team)

To secure services:

✅ Only Administrators should have SERVICE_CHANGE_CONFIG
✅ Remove Authenticated Users from service ACL
✅ Restrict file system write access
✅ Monitor service config changes
✅ Enable logging for Event ID 7045 (new service installed)
✅ Monitor changes to binary path

Check via:

```
services.msc
```

Or security auditing tools.

---

# 9️⃣ Real-World Notes (Modern Windows)

Modern systems are usually secure.

Most vulnerabilities appear in:

* Third-party enterprise software
* Backup software
* Monitoring tools
* Legacy applications
* Custom in-house services

---

# 10️⃣ Quick Attack Decision Tree

When you see a service:

1️⃣ Does it run as SYSTEM?
2️⃣ Can I modify its executable file?
3️⃣ Can I modify its folder?
4️⃣ Can I change its config?
5️⃣ Can I start/stop it?

If yes to any combination → possible escalation.

