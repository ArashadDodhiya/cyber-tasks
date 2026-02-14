# 📘 Attack Chain: From Low Privilege to Domain Escalation (AD Lab)

This attack chain demonstrates how a tester can escalate privileges inside a Windows Active Directory environment by:

1. Getting low-privilege access
2. Dumping NTLM hashes
3. Cracking the hash
4. Reusing credentials for lateral movement
5. Escalating to Domain Admin

---

# 🧱 Stage 1 – Initial Low-Privilege Access

This is your entry point.

### Possible Initial Access Methods

* Phishing attack
* Exploiting vulnerable web app
* Weak SSH/SMB credentials
* Password spraying
* Captured NetNTLMv2 via Responder
* Local file inclusion leading to shell

End result:

```
You have a shell as a normal domain user.
Example:
DOMAIN\john
```

This user:

* Is NOT admin
* Has limited permissions
* Can log into one machine

But that’s enough to start.

---

# 🧱 Stage 2 – Privilege Context & Why It Matters

On Windows systems:

Credentials are stored in memory when users log in.

Specifically:

* Inside the LSASS process
* In SAM database (local accounts)
* In NTDS.dit (domain controller)

If your compromised machine has:

* An administrator logged in
* Or cached domain credentials

You can extract those credentials.

---

# 🧱 Stage 3 – Running Mimikatz

Tool used:

Mimikatz

### Goal:

Dump password hashes or plaintext credentials from memory.

---

## Required Condition

You must have:

* Local admin privileges
  OR
* SYSTEM privileges

If you only have normal user, escalation is needed first (via privilege escalation vulnerability).

---

## Core Commands

Enable debug privilege:

```
privilege::debug
```

Dump credentials:

```
sekurlsa::logonpasswords
```

Example output:

```
Username: Administrator
Domain  : DOMAIN
NTLM    : 8846f7eaee8fb117ad06bdd830b7586c
```

Now you have the NTLM hash.

---

# 🧱 Stage 4 – Understanding NTLM Hash

NTLM is:

* Fast hashing algorithm
* Unsalted
* Stored for Windows authentication

Example NTLM hash:

```
8846f7eaee8fb117ad06bdd830b7586c
```

This is NOT reversible.
But it is crackable via brute force or dictionary attack.

---

# 🧱 Stage 5 – Cracking the NTLM Hash

Tool used: Hashcat

```
hashcat -m 1000 -a 0 hash.txt rockyou.txt
```

### Why `-m 1000`?

Mode 1000 = NTLM

### What Happens Internally?

1. Hashcat takes password from wordlist
2. Converts to NTLM
3. Compares with dumped hash
4. If match → password cracked

Example result:

```
8846f7eaee8fb117ad06bdd830b7586c:Password@123
```

Now you have plaintext credentials.

---

# 🧱 Stage 6 – Credential Reuse (Lateral Movement)

Most organizations reuse passwords.

This is where attackers win.

---

## 🔹 RDP Access

If Administrator uses same password:

```
RDP login:
Username: Administrator
Password: Password@123
```

Now you get full desktop access.

---

## 🔹 SMB Access

Access file shares:

```
\\192.168.1.10\C$
```

If credentials valid → you gain administrative access to file system.

---

# 🧱 Stage 7 – Domain Escalation

Now things get serious.

If the cracked credentials belong to:

* Domain Admin
* Server Admin
* Backup Operator
* Service Account with high privileges

You can:

* Access Domain Controller
* Dump entire AD database
* Create new admin user
* Modify group memberships

---

## Typical Escalation Path

1. Dump local admin hash
2. Crack password
3. Reuse password on Domain Controller
4. Gain Domain Admin access
5. Dump NTDS.dit
6. Extract all domain hashes

---

# 🧠 Why This Chain Works

Because of:

* Weak passwords
* Password reuse
* Cached credentials
* Lack of credential guard
* Overprivileged accounts

---

# 🔥 Visual Flow of the Attack

```
Initial Access (low user)
        ↓
Privilege Escalation (local admin)
        ↓
Run Mimikatz
        ↓
Dump NTLM hash
        ↓
Crack hash (Hashcat -m 1000)
        ↓
Reuse password (RDP / SMB)
        ↓
Lateral movement
        ↓
Domain Admin compromise
```

---

# ⚠️ Important Alternative: Pass-the-Hash

You don’t always need to crack.

If NTLM hash is valid, you can authenticate using hash directly.

Example tools:

* Impacket
* Evil-WinRM
* PsExec

This is called:
**Pass-the-Hash (PtH)**

Cracking is needed only when:

* You want plaintext
* You want to reuse password elsewhere
* Hash-only auth not possible

---

# 🛡 Defensive Controls That Break This Chain

Modern defenses:

* Credential Guard
* LSASS protection
* LAPS (Local Admin Password Solution)
* Strong password policies
* MFA
* Network segmentation
* Tiered admin model
* Restricted admin mode for RDP

---

# 🎯 Key Learning Insight

The real power isn’t Mimikatz.

It’s:

* Credential reuse
* Weak passwords
* Human behavior

In real pentests, 70% of AD compromises happen because of:

* Poor password hygiene
* Privileged accounts logging into workstations
