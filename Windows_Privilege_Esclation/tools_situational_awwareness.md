# 🔍 First: What Do We Mean by Active vs Passive?

### 🟢 Passive Enumeration

* No interaction with the target system
* No new processes spawned
* No registry queries triggered by you
* No file system scanning initiated by you

Example:

* Reading already available output
* Observing environment variables
* Checking your current token

---

### 🔴 Active Enumeration

* Executes commands
* Queries registry
* Scans filesystem
* Enumerates services
* Uses WMI, PowerShell, .NET reflection
* Touches lots of objects quickly

This creates:

* Event logs
* PowerShell logs
* EDR telemetry
* AV detections

---

# 🛠 Tool Breakdown

## 1️⃣ WinPEAS

### Nature: 🔴 Highly Active

What it does:

* Scans registry
* Enumerates services
* Checks permissions on many folders
* Queries scheduled tasks
* Searches for passwords
* Enumerates network configs

It touches **hundreds to thousands of objects**.

### Detection Likelihood: HIGH

* Triggers Defender often
* Very noisy in EDR
* Large number of file/registry queries
* Often signature-based detection

Blue team will see:

* Massive enumeration activity
* Suspicious binary execution
* Potential alert on known hash/signature

---

## 2️⃣ Seatbelt

### Nature: 🔴 Active (but can be less noisy)

Seatbelt uses:

* .NET API calls
* WMI queries
* Registry enumeration
* Security descriptor checks

It’s modular — you can run specific checks instead of everything.

### Detection Likelihood: MEDIUM–HIGH

* WMI logging (Event ID 5861 etc.)
* PowerShell logs (if loaded reflectively)
* .NET suspicious assembly load
* EDR behavioral detection

Less noisy than WinPEAS if you use specific modules only.

---

## 3️⃣ PowerUp (PowerShell)

### Nature: 🔴 Active

PowerUp:

* Enumerates services
* Checks ACLs
* Queries registry
* Checks AlwaysInstallElevated
* Scans writable directories

### Detection Likelihood: HIGH

Because:

* PowerShell is heavily monitored
* Script block logging (Event ID 4104)
* AMSI scanning
* EDR behavioral analytics

Blue team can see:

* Suspicious PowerShell functions
* Privilege escalation keyword patterns
* Encoded command usage

---

## 4️⃣ SharpUp

### Nature: 🔴 Active (but lightweight)

SharpUp:

* Checks service permissions
* Checks unquoted service paths
* Registry misconfigs
* Writable paths

Less comprehensive than WinPEAS.

### Detection Likelihood: MEDIUM

Still:

* Binary execution logged
* Behavioral patterns detected

---

# 🧠 So Will the Owner Know?

## 🟡 In a Home Lab:

No EDR → Probably not.

## 🟠 In Corporate Environment:

Yes, likely.

Modern detection stack monitors:

* Process creation (Event ID 4688)
* PowerShell logs
* AMSI scans
* Registry access
* Suspicious privilege enumeration
* Known tool signatures

Even if not signature-detected:
The *behavior* is noisy.

---

# 📊 Noise Comparison

| Tool     | Noise Level    | AV Trigger  | EDR Detection |
| -------- | -------------- | ----------- | ------------- |
| WinPEAS  | 🔴 Very High   | Very Likely | Very Likely   |
| PowerUp  | 🔴 High        | Likely      | Very Likely   |
| Seatbelt | 🟠 Medium–High | Possible    | Likely        |
| SharpUp  | 🟠 Medium      | Possible    | Possible      |

---

# 🛡 Blue Team Perspective

These tools trigger alerts because they:

* Enumerate many services quickly
* Query registry keys related to privilege escalation
* Access sensitive locations in bulk
* Run from suspicious paths (e.g., Downloads, Temp)

Defenders can detect:

* Enumeration burst activity
* Known offensive tool strings
* Suspicious command-line arguments

---

# 🔥 Mature Operator Mindset

In real red team engagements:

1. Do manual enumeration first.
2. Run targeted checks.
3. Avoid “scan everything” mode.
4. Limit modules.
5. Rename binaries doesn’t bypass behavior detection.

---

# 🎯 Important Takeaway

These tools are:

> Not stealth tools.
> They are convenience enumeration frameworks.

They are designed for:

* Labs
* CTF
* Internal red team with controlled detection testing

Not stealthy APT simulation by default.