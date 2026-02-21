# 🧠 First: Where Cached Credentials Live

Cached AD credentials (MSCache / DCC2 hashes) are stored:

* In the **registry hive**:
  `HKLM\Security\Cache`
* Protected by the **SYSTEM account**
* Encrypted using the **SYSTEM boot key**

So an attacker must first gain **high privileges (local admin or SYSTEM)** before attempting extraction.

---

# 🔥 High-Level Attack Flow

Here’s how attackers typically dump cached credentials in a real intrusion:

---

## Step 1️⃣ – Gain Local Admin Access

Attackers first compromise a machine via:

* Phishing → malware execution
* Exploiting a vulnerable service
* Credential reuse
* Privilege escalation vulnerability

They need **local administrator rights**.

Without admin access → cached credentials cannot be dumped.

---

## Step 2️⃣ – Access Protected System Files / Registry

Cached credentials are stored inside the **SECURITY registry hive**, which is locked while Windows is running.

Attackers typically:

* Access the registry directly with SYSTEM privileges
* Or create a copy of the registry hives for offline extraction

Relevant hives:

* `SAM`
* `SYSTEM`
* `SECURITY`

The **SYSTEM hive contains the boot key**, which is required to decrypt cached credentials.

---

## Step 3️⃣ – Extract MSCache (DCC2) Hashes

Using credential-dumping tools (in authorized red team environments), attackers:

* Parse the SECURITY hive
* Decrypt stored cached credentials
* Extract **DCC2 hashes**

These are not plaintext passwords — they are hashed versions.

---

## Step 4️⃣ – Offline Cracking

Since MSCache hashes can’t usually be reused directly:

Attackers attempt:

* Dictionary attacks
* Wordlist attacks
* Rule-based cracking

If passwords are weak → they may recover plaintext credentials.

---

# ⚠️ Important Difference

Cached credentials are:

* ❌ Not in LSASS memory (like NTLM/Kerberos tickets)
* ❌ Not directly usable for Pass-the-Hash
* ✅ Usable for offline cracking

So they are less immediately dangerous than LSASS credential dumping — but still serious.

---

# 🧪 Example Attack Scenario

Let’s say:

1. A remote employee logs into a domain-joined laptop.
2. Their credentials are cached.
3. Later, malware infects the laptop.
4. Attacker gains local admin.
5. Attacker extracts cached hashes.
6. The employee’s password is weak (e.g., `Winter2024!`).
7. Password is cracked.
8. Attacker now has valid domain credentials.

If that employee had elevated privileges → lateral movement becomes possible.

---

# 🛡️ Defensive Perspective: How to Detect It

### 🚨 Indicators of Credential Dumping

Look for:

* Unusual access to:

  * `HKLM\Security`
  * `HKLM\System`
* Suspicious creation of:

  * Registry hive copies
* Event logs showing:

  * Privilege escalation
  * SYSTEM-level process creation
* EDR alerts involving:

  * Credential access behavior
  * Registry dumping
  * LSASS or SAM access attempts

MITRE ATT&CK mapping:

* **T1003 – OS Credential Dumping**
* Sub-technique: Cached Domain Credentials

---

# 🔐 How to Defend Against Cached Credential Dumping

## 1️⃣ Reduce Cached Logon Count

Set:

```
Interactive logon: Number of previous logons to cache = 0
```

(Only in high-security environments — may affect remote workers.)

---

## 2️⃣ Use Strong Password Policies

Because cached credentials must be cracked offline:

* Long passwords (14+ chars)
* Passphrases
* No predictable patterns

---

## 3️⃣ Enable Credential Guard

Helps protect against credential theft and limits exposure.

---

## 4️⃣ Prevent Privileged Account Logins to Workstations

Never allow:

* Domain Admin accounts
* Enterprise Admin accounts
* Tier 0 accounts

To log into normal user machines.

---

## 5️⃣ Use LAPS

Ensures local admin passwords are:

* Unique per machine
* Rotated regularly

Prevents lateral movement.

---

# 🎯 Key Takeaway

Attackers dump cached credentials by:

1. Gaining admin access
2. Extracting encrypted registry data
3. Decrypting MSCache hashes
4. Attempting offline cracking

The real risk comes from:

> Weak passwords + high-privilege accounts logging into lower-tier systems.

---

If you’d like, I can next explain:

* 🔍 Difference between LSASS dumping vs cached credential dumping
* 🛡️ How Credential Guard technically protects credentials
* 📊 How to detect this in Windows Event Logs
* 🧱 Tiered AD architecture (Tier 0/1/2 model)

Which direction would you like to go?
