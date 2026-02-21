# 🧠 What Are Cached AD Credentials?

When a user logs into a **Windows computer that is joined to an Active Directory (AD) domain**, the computer usually checks with a **Domain Controller (DC)** to verify the username and password.

But what if:

* The laptop is **not connected to the company network**?
* The domain controller is **offline**?
* The user is working remotely?

To solve this, Windows **stores (caches) a secure version of the user’s credentials locally** after a successful login.

This allows the user to log in again even without contacting the domain controller.

---

# 🏢 Simple Real-World Example

Imagine:

* You work at a company.
* Your laptop is joined to the company domain.
* You log in at the office (connected to the network).

👉 Windows verifies your password with the domain controller
👉 Then it **stores a hashed version of your credentials locally**

Now later:

* You travel home.
* You are NOT connected to the company VPN.
* You turn on your laptop and log in.

Even though the domain controller is unreachable:

✅ You can still log in
Because Windows uses the **cached credentials** stored locally.

---

# 🔐 What Exactly Is Stored?

Windows does **NOT store your plain-text password**.

Instead, it stores:

* A hashed version of your password
* Using a special format called:

  * **MSCache**
  * Also known as **DCC (Domain Cached Credentials)**

This is stored in:

```
C:\Windows\System32\config\SAM
```

Technically, the hash is stored in:

```
HKLM\Security\Cache
```

But it's protected and requires SYSTEM-level access to read.

---

# ⚙️ How Cached Credentials Work (Step-by-Step)

### First Login (Online)

1. User enters username and password.
2. Windows sends credentials to Domain Controller.
3. DC verifies them.
4. Windows creates and stores a cached hash locally.
5. User logs in.

---

### Second Login (Offline)

1. User enters username and password.
2. Windows hashes the entered password.
3. Windows compares it to the locally stored cached hash.
4. If it matches → login successful.
5. No domain controller needed.

---

# 🔢 How Many Credentials Are Cached?

By default:

Windows caches credentials for the **last 10 users** who logged in.

You can check or configure this via:

```
Interactive logon: Number of previous logons to cache
```

Location:

```
Local Security Policy → Security Options
```

Registry key:

```
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
Value: CachedLogonsCount
```

---

# 🔥 Why Is This Important for Security?

Cached credentials are useful — but they also introduce risk.

If an attacker:

* Gains local admin access
* Dumps the cached credential hashes

They may attempt:

* **Offline password cracking**
* Password reuse attacks

This is commonly done using tools like:

* Mimikatz
* Hashcat (for offline cracking)
* secretsdump.py (in certain scenarios)

⚠️ Ethical note: These tools should only be used in authorized penetration testing environments.

---

# 🛑 Important Difference: Cached Credentials vs NTLM Hash

Cached credentials (MSCache/DCC) are:

* NOT the same as NTLM hashes
* Harder to reuse directly for Pass-the-Hash
* Must usually be cracked offline

So they’re less immediately exploitable than NTLM hashes — but still sensitive.

---

# 🧪 Ethical Hacking Example Scenario

### Scenario:

You're performing an internal penetration test.

You compromise a domain-joined laptop that was used remotely.

You discover:

* The machine has cached credentials.
* A domain admin logged into it previously.

Now you:

1. Extract cached credential hashes.
2. Attempt offline cracking.
3. If password is weak → recover clear-text password.
4. Use recovered credentials to move laterally.

This is why:

👉 Strong passwords matter
👉 Limiting privileged logins to workstations is critical
👉 Credential Guard and LAPS are important defenses

---

# 🛡️ Defensive Best Practices

To reduce risk:

### 1️⃣ Reduce cached logon count

Set it to:

```
0 (if high security environment)
```

### 2️⃣ Use strong password policies

### 3️⃣ Enable Credential Guard

Prevents credential theft from memory.

### 4️⃣ Avoid logging into workstations with Domain Admin accounts

### 5️⃣ Use LAPS

Ensures local admin passwords are unique.

---

# 🧠 Quick Comparison Table

| Feature                     | Cached Credentials         |
| --------------------------- | -------------------------- |
| Stored locally?             | ✅ Yes                      |
| Plain-text password stored? | ❌ No                       |
| Can login offline?          | ✅ Yes                      |
| Can attacker extract?       | Possible with admin access |
| Can be cracked?             | Yes (offline attack)       |

---

# 🎯 Key Takeaway (Simple Version)

Cached AD credentials are:

> A locally stored, hashed backup of your domain login that allows you to log in without network access.

They are helpful for usability but create a potential security risk if a machine is compromised.

---

If you’d like, I can next explain:

* 🔎 How attackers dump cached credentials (high-level)
* 🛡️ How Credential Guard protects against this
* 🧪 How to detect cached credential attacks in logs
* 🔬 Difference between LSASS memory credentials vs cached credentials

Just tell me which direction you want to go.
