# 🔴 SMB Enumeration (Active Information Gathering)

## What is SMB?

**SMB (Server Message Block)** is a network file-sharing protocol used mainly on **Windows systems**.

### Common SMB ports

* **139** → NetBIOS Session Service (older SMB)
* **445** → SMB over TCP (modern, most important)

---

## 🎯 Goal of SMB Enumeration

SMB enumeration helps you find:

* Live Windows hosts
* OS version
* Computer name
* Domain / Workgroup
* SMB services & shares
* Misconfigurations (null sessions, weak auth)

---

# 1️⃣ Port Scan for SMB (Corrected)

### Your note (fixed)

```bash
nmap -v -p 139,445 -oG smb.txt <target>
```

### Explanation

* `-v` → verbose output
* `-p 139,445` → scan SMB ports
* `-oG smb.txt` → save grepable output
* `<target>` → IP or network

### What you learn

* Which hosts have SMB open
* First step before SMB-specific scans

---

# 2️⃣ SMB Network Scan (Range Scan)

### Your note (fixed)

```bash
nmap -v -p 139,445 192.168.50.0/24
```

### Explanation

* Scans **entire subnet**
* Finds all machines exposing SMB
* Used for **internal network enumeration**

📌 Very common in internal pentests & labs

---

# 3️⃣ SMB OS Discovery (MOST IMPORTANT)

### Your note (fixed)

```bash
nmap -v -p 139,445 --script smb-os-discovery <target>
```

### Explanation

Uses **Nmap NSE (Scripting Engine)** to query SMB for system info.

---

## 🧠 What `smb-os-discovery` reveals

If SMB allows it, you may get:

* OS version (Windows 7 / 10 / Server)
* Computer name
* Domain or workgroup
* SMB version
* System time

### Example output

```text
OS: Windows 10 Pro 19045
Computer name: FILE-SERVER
Domain: WORKGROUP
```

🔥 Extremely valuable for attackers
🔥 Also very visible in logs

---

# 🔔 Why SMB Enumeration is Dangerous (for defenders)

Because SMB:

* Often misconfigured internally
* Trusted inside networks
* Rich in metadata

SOC often sees:

```text
SMB Session Setup
Tree Connect Requests
Null session attempts
```

---

# 🧪 Typical SMB Enumeration Flow

```text
1. Scan ports 139/445
2. Identify live SMB hosts
3. Run smb-os-discovery
4. Enumerate shares (later)
5. Check authentication weaknesses
```

---

# ⚠️ Legal & Practical Warning

> SMB enumeration should **only** be done on:

* Authorized internal networks
* Labs (HTB, TryHackMe)
* Explicit pentest scopes

Unauthorized SMB scanning = **high-risk activity**

---

# 🧠 One-Line Takeaway

> **SMB enumeration exposes Windows system details by actively querying ports 139 and 445 — making it powerful but highly detectable.**
