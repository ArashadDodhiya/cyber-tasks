⚠️ Important: UAC bypass ≠ gaining admin from a normal user.
It usually works when the attacker is **already a local administrator** but wants to execute without a UAC prompt.

---

# 🧱 First: What Is UAC?

### UAC = User Account Control

That popup you see:

> “Do you want to allow this app to make changes to your device?”

That’s UAC.

---

# 🧠 How UAC Actually Works (Simple Explanation)

Even if you're in the **Administrators group**, Windows does NOT run you as full admin all the time.

Instead:

When an admin logs in, Windows creates **two tokens**:

1. 🟢 Standard user token (used by default)
2. 🔴 Full admin token (used only after UAC approval)

So even if you’re admin, you're running as **standard user most of the time**.

When you click “Yes” on UAC:
→ Windows switches from the limited token to the full admin token.

---

# 🎯 What Is a UAC Bypass?

A UAC bypass is when:

> A process gets the full admin token WITHOUT showing the UAC prompt.

Important:

* It does NOT turn a normal user into admin.
* It bypasses the UAC popup for someone already in Administrators.

---

# 🔥 Why Do UAC Bypasses Exist?

Because:

* Some Windows programs are marked as “auto-elevate”
* Windows trusts certain system binaries
* Some registry locations are writable by users
* Windows loads configuration from user-controlled locations

Attackers abuse this trust.

---

# 🧩 Core Idea Behind Most UAC Bypasses

Most UAC bypasses abuse one of these:

1. Auto-elevating executables
2. DLL hijacking
3. Registry hijacking
4. Environment variable hijacking
5. COM object abuse

The pattern is usually:

1. Find trusted auto-elevated binary
2. Trick it into loading attacker-controlled code
3. It runs with high integrity
4. No UAC popup

---

# 📌 What Is “Auto-Elevate”?

Some Microsoft-signed binaries are marked in their manifest as:

```
<autoElevate>true</autoElevate>
```

That means:

If the user is an Administrator → run elevated automatically
WITHOUT showing UAC prompt.

Windows assumes these are safe.

That assumption can be abused.

---

# 🧪 Simple Conceptual Example (Registry-Based UAC Bypass)

Let’s explain one classic method conceptually.

⚠️ This is explanation only — not a step-by-step exploit guide.

---

## Step 1: Find an Auto-Elevated Binary

Example:

* fodhelper.exe
* computerdefaults.exe
* sdclt.exe (older)
* eventvwr.exe (older technique)

These are trusted Microsoft binaries.

---

## Step 2: Understand What It Loads

Some of these programs:

* Look up registry keys
* Load handlers
* Launch commands based on registry values

Problem:
Some registry locations are writable by normal users.

---

## Step 3: Registry Hijacking Concept

If:

Trusted auto-elevated program reads:

```
HKCU\Software\Classes\SomeKey\Shell\Open\Command
```

And that key doesn’t exist…

Windows might allow user to create it.

So attacker:

* Creates that registry key
* Points it to malicious executable

---

## Step 4: Launch Auto-Elevated Binary

When the trusted binary runs:

* It reads the hijacked registry key
* Executes attacker’s command
* Runs it elevated
* No UAC prompt appears

Boom 💥

You now have high-integrity process.

---

# 🧠 Why Does This Work?

Because:

* Windows trusts signed Microsoft binaries
* It assumes registry values are safe
* It auto-elevates certain programs
* It doesn’t validate some user-writable paths properly

This is a design trust issue.

---

# 🔍 How Integrity Levels Work

Windows has integrity levels:

* Low
* Medium (normal user/admin default)
* High (elevated admin)
* System

UAC bypass moves you from:

Medium → High

It does NOT give SYSTEM.

For SYSTEM you need another PrivEsc.

---

# 🚫 Common Misunderstanding

❌ UAC bypass does NOT work if:

* User is not in Administrators group
* UAC is set to “Always Notify” (some methods fail)
* You're already standard user only

---

# 🧰 Common UAC Bypass Techniques (Conceptual)

| Technique Type             | Example Concept            |
| -------------------------- | -------------------------- |
| Registry hijack            | Override handler key       |
| DLL hijack                 | Place DLL in search path   |
| COM hijack                 | Override COM object        |
| Environment variable abuse | Manipulate system variable |
| Scheduled task abuse       | Auto-elevated task         |

---

# 🏢 Real-World Scenario

Example attack chain:

1. Phishing attack
2. User is local administrator
3. Malware runs (medium integrity)
4. Malware performs UAC bypass
5. Gains high integrity
6. Disables AV
7. Dumps credentials
8. Moves laterally

UAC bypass is usually a mid-stage technique.

---

# 🛡 Blue Team Detection

Monitor:

* Auto-elevated binaries spawning unusual processes
* Registry modifications in:

  * HKCU\Software\Classes
* High integrity processes started by medium processes
* Unusual parent-child process relationships

Good tools:

* Sysmon
* Windows Event Logs
* EDR solutions

---

# 🔐 How To Prevent UAC Bypass

1. Set UAC to “Always Notify”
2. Remove local admin rights from users
3. Use least privilege model
4. Monitor registry modifications
5. Use application control (AppLocker, WDAC)
6. Keep Windows updated

Most bypasses get patched over time.

---

# 🎯 Key Takeaways

* UAC bypass ≠ privilege escalation from normal user
* It bypasses the UAC popup
* Requires user already in Administrators group
* Exploits trust in auto-elevated binaries
* Used as part of larger attack chains

---

# 🧪 Lab Practice (Safe Way)

If you want to study this safely:

1. Create Windows VM
2. Create admin user
3. Turn UAC to default
4. Monitor with Process Monitor
5. Observe integrity levels
6. Study registry lookups

Focus on understanding behavior, not just running tools.

---

If you'd like next, I can explain:

* 🔬 Difference between UAC bypass vs real PrivEsc
* 🧬 How tokens & integrity levels work internally
* 🛡 Detection engineering for UAC bypass
* 📚 A structured lab roadmap for mastering Windows PrivEsc

What level are you aiming for — OSCP-style, CRTO-style, or enterprise red team?
