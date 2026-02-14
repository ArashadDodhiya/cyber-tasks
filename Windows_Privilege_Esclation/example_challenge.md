# 🖥️ Scenario: “Helpdesk Gone Wrong”

## Environment

* Machine: `WIN10-CLIENT`
* Joined to domain: `corp.local`
* Your access:

  ```
  Username: helpdesk
  Password: Helpdesk@2025
  ```
* Access method: RDP
* Goal: Get **Administrator** on the local machine

---

# ⚠️ Rules (OSCP Style)

* No Metasploit
* No exploit-db
* No public exploit scripts
* Use built-in tools only
* Assume you have command line access

---

# 🔎 Phase 1 – Situational Awareness

After logging in:

```cmd
whoami
whoami /groups
whoami /priv
```

Findings:

* User: `corp\helpdesk`
* Member of:

  * Domain Users
  * Remote Desktop Users
* No special privileges enabled

You are NOT local admin.

---

# 🔍 Phase 2 – Local Enumeration

Check local admins:

```cmd
net localgroup administrators
```

Output:

```
Administrator
ITAdmin
```

You are not listed.

---

# 🔍 Phase 3 – Service Enumeration

```cmd
wmic service get name,displayname,pathname,startmode
```

You notice something interesting:

```
DisplayName: Inventory Service
PathName: C:\Program Files Inventory System\inventory.exe
StartMode: Auto
```

⚠️ The path:

```
C:\Program Files Inventory System\inventory.exe
```

It has spaces and **no quotes**.

Classic.

---

# 🔍 Phase 4 – Check Folder Permissions

```cmd
icacls "C:\Program Files Inventory System"
```

Output:

```
BUILTIN\Users:(M)
```

🔥 That means:

* Any user can MODIFY files inside this folder.

This is major.

---

# 🧠 What You Should Be Thinking

* Service runs as SYSTEM?
* Path is unquoted?
* Folder writable?
* Can I replace the binary?

---

# 🔎 Phase 5 – Confirm Service Context

```cmd
sc qc "Inventory Service"
```

Output:

```
SERVICE_START_NAME: LocalSystem
```

💥 The service runs as SYSTEM.

Now you have:

| Condition          | Status |
| ------------------ | ------ |
| Runs as SYSTEM     | ✅      |
| Writable directory | ✅      |
| Auto start         | ✅      |
| Unquoted path      | ✅      |

This is a privilege escalation vector.

---

# 🧩 But Here’s the Twist

When you try to restart the service:

```cmd
net stop "Inventory Service"
```

You get:

```
Access is denied.
```

You cannot restart services.

So now what?

This is where OSCP thinking begins.

---

# 🔎 Phase 6 – Check If You Can Reboot the Machine

```cmd
whoami /priv
```

You notice:

```
SeShutdownPrivilege
```

It is ENABLED.

Interesting…

Even standard users sometimes have shutdown rights.

---

# 🧠 Chain the Logic

You cannot restart the service manually.

But:

* Service is set to Auto
* It runs as SYSTEM
* Folder is writable
* You can reboot

So…

What happens on reboot?

All Auto services start.

---

# 🎯 The Exploitation Path (Conceptual)

The attack chain is:

1. Replace `inventory.exe` with your controlled executable
2. Reboot machine
3. Service starts as SYSTEM
4. Your binary executes as SYSTEM

This is a realistic OSCP-style path.

---

# 🔎 Bonus Enumeration (Hidden in Plain View #2)

While searching:

```powershell
Get-ChildItem -Path C:\ -Recurse -Include *.config,*.xml -ErrorAction SilentlyContinue
```

You also find:

```
C:\Program Files Inventory System\web.config
```

You check:

```cmd
type "C:\Program Files Inventory System\web.config"
```

You see:

```xml
<add key="dbUser" value="sa"/>
<add key="dbPass" value="SuperSecure123"/>
```

Now you potentially have:

* SQL credentials
* Maybe reused passwords
* Maybe domain password reuse

OSCP loves layered enumeration.

---

# 🧠 What This Challenge Teaches

This is pure exam-style thinking:

✔ Don’t jump to exploit
✔ Enumerate carefully
✔ Check permissions
✔ Read configs
✔ Think about service behavior
✔ Chain weaknesses

---

# 🛡️ Blue Team Lessons

This vulnerability exists because:

* Developers installed software incorrectly
* NTFS permissions were loosened
* Service paths weren’t quoted
* Credentials stored in config
* No configuration review

---

# 🧪 How To Build This Lab

1. Create Windows VM
2. Create standard user `helpdesk`
3. Create folder:

   ```
   C:\Program Files Inventory System
   ```
4. Grant Users Modify:

   ```cmd
   icacls "C:\Program Files Inventory System" /grant Users:(M)
   ```
5. Create dummy service pointing to that path
6. Set service to Auto and LocalSystem
7. Do NOT quote path

Now practice full enumeration.

---

# 🎓 OSCP Exam Strategy Reminder

When stuck, always check:

* Services
* Scheduled tasks
* Writable directories
* Registry autoruns
* Startup folders
* Config files
* Token privileges
* Stored credentials