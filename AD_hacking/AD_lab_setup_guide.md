# 🏠 Active Directory Lab Setup Guide for Ethical Hacking

This guide walks you through setting up a **complete AD hacking lab** at home for legal practice. Every technique in the other documents can be practiced safely in this environment.

> ⚠ This lab is for **educational and authorized testing purposes only**. Never attack systems without explicit written permission.

---

## 🧠 Why Build an AD Lab?

* Practice every AD attack technique safely
* Understand both red team and blue team perspectives
* Prepare for certifications (OSCP, HTB CPTS, CRTP, eCPPT)
* Test detection rules and tools
* No risk of legal consequences

---

## 🖥 Hardware Requirements

| Component | Minimum | Recommended |
| --- | --- | --- |
| RAM | 16 GB | 32 GB |
| CPU | 4 cores | 8 cores |
| Storage | 100 GB free | 250 GB SSD |
| Virtualization | VirtualBox (free) | VMware Workstation Pro |

---

## 🏗 Lab Architecture

```text
┌──────────────────────────────────────────────┐
│                  NAT Network                 │
│               192.168.10.0/24                │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   DC01   │  │   WS01   │  │   WS02   │  │
│  │ Win Srv  │  │  Win 10  │  │  Win 10  │  │
│  │ .10      │  │  .50     │  │  .51     │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                              │
│  ┌──────────┐  ┌──────────┐                 │
│  │  SRV01   │  │  Kali    │                 │
│  │ Win Srv  │  │  Linux   │                 │
│  │  .20     │  │  .100    │                 │
│  └──────────┘  └──────────┘                 │
└──────────────────────────────────────────────┘
```

---

## 📦 Virtual Machines Needed

| VM | OS | Role | IP |
| --- | --- | --- | --- |
| DC01 | Windows Server 2019/2022 | Domain Controller | 192.168.10.10 |
| SRV01 | Windows Server 2019 | Member Server (file/SQL) | 192.168.10.20 |
| WS01 | Windows 10 Enterprise | Workstation 1 | 192.168.10.50 |
| WS02 | Windows 10 Enterprise | Workstation 2 | 192.168.10.51 |
| Kali | Kali Linux | Attacker Machine | 192.168.10.100 |

---

## 🔧 Step-by-Step Lab Setup

---

### Step 1: Download ISOs

* **Windows Server 2019/2022 Evaluation** (free 180-day trial):
  [https://www.microsoft.com/en-us/evalcenter/](https://www.microsoft.com/en-us/evalcenter/)

* **Windows 10 Enterprise** (free 90-day trial):
  [https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise)

* **Kali Linux**:
  [https://www.kali.org/get-kali/](https://www.kali.org/get-kali/)

---

### Step 2: Create Network in VirtualBox/VMware

#### VirtualBox

1. File → Host Network Manager → Create
2. Set IP: `192.168.10.1`, Mask: `255.255.255.0`
3. Disable DHCP
4. Assign all VMs to this Host-Only network

#### VMware

1. Edit → Virtual Network Editor
2. Add Network → VMnet2 (Host-Only)
3. Subnet: `192.168.10.0`, Mask: `255.255.255.0`
4. No DHCP

---

### Step 3: Install & Configure DC01 (Domain Controller)

1. Install Windows Server 2019/2022
2. Set static IP:

   ```
   IP: 192.168.10.10
   Subnet: 255.255.255.0
   Gateway: 192.168.10.1
   DNS: 127.0.0.1
   ```

3. Rename computer:

   ```powershell
   Rename-Computer -NewName "DC01" -Restart
   ```

4. Install Active Directory:

   ```powershell
   Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
   ```

5. Promote to Domain Controller:

   ```powershell
   Install-ADDSForest -DomainName "corp.local" -DomainNetbiosName "CORP" -InstallDns:$true
   ```

6. Set DSRM password when prompted. Server will restart.

---

### Step 4: Create AD Users & Groups

```powershell
# Create OUs
New-ADOrganizationalUnit -Name "IT" -Path "DC=corp,DC=local"
New-ADOrganizationalUnit -Name "HR" -Path "DC=corp,DC=local"

# Create Users
New-ADUser -Name "John Smith" -SamAccountName "jsmith" -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true -Path "OU=IT,DC=corp,DC=local"

New-ADUser -Name "Jane Doe" -SamAccountName "jdoe" -AccountPassword (ConvertTo-SecureString "Welcome2025!" -AsPlainText -Force) -Enabled $true -Path "OU=HR,DC=corp,DC=local"

New-ADUser -Name "SQL Service" -SamAccountName "sqlsvc" -AccountPassword (ConvertTo-SecureString "SQLAdmin2020!" -AsPlainText -Force) -Enabled $true -Path "OU=IT,DC=corp,DC=local"

New-ADUser -Name "IT Admin" -SamAccountName "itadmin" -AccountPassword (ConvertTo-SecureString "ITAdmin@2025" -AsPlainText -Force) -Enabled $true -Path "OU=IT,DC=corp,DC=local"

# Add Domain Admin
Add-ADGroupMember -Identity "Domain Admins" -Members "itadmin"

# Set SPN for Kerberoasting practice
Set-ADUser -Identity "sqlsvc" -ServicePrincipalNames @{Add="MSSQLSvc/srv01.corp.local:1433"}

# Disable pre-auth for AS-REP Roasting practice
Set-ADAccountControl -Identity "jdoe" -DoesNotRequirePreAuth $true
```

---

### Step 5: Configure Vulnerable Settings

#### Enable WDigest (Cleartext passwords in LSASS)

```powershell
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f
```

#### Disable Windows Defender (Lab only!)

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

#### Create a Shared Folder with GPP Password

```powershell
New-SmbShare -Name "Share" -Path "C:\Share" -FullAccess "Everyone"
```

#### Add Local Admin on Workstations (Simulate reused admin password)

On WS01 and WS02 after joining domain:

```powershell
net user localadmin Password123! /add
net localgroup administrators localadmin /add
```

---

### Step 6: Join Workstations & Server to Domain

On each Windows 10/Server machine:

1. Set DNS to `192.168.10.10`
2. Join domain:

   ```powershell
   Add-Computer -DomainName "corp.local" -Credential CORP\Administrator -Restart
   ```

---

### Step 7: Simulate Logins (For Credential Harvesting)

Log into WS01 as `itadmin` (Domain Admin) — this caches credentials:

```
Username: CORP\itadmin
Password: ITAdmin@2025
```

Log into WS02 as `jsmith`:

```
Username: CORP\jsmith
Password: Password123!
```

Now LSASS on WS01 has Domain Admin credentials cached.

---

### Step 8: Setup Kali Linux (Attacker)

```bash
# Set static IP
sudo ip addr add 192.168.10.100/24 dev eth0

# Install key tools
sudo apt update
sudo apt install -y bloodhound neo4j crackmapexec responder impacket-scripts
```

---

## 🧪 What to Practice

After setup, practice these attacks:

| Attack | Document |
| --- | --- |
| LLMNR/NBT-NS Poisoning | `LLMNR_NBNS_poisoning.md` |
| Password Spraying | `AD_password_attacks.md` |
| AS-REP Roasting | `AS_REP_roasting.md` |
| Kerberoasting | `kerberoasting.md` |
| Credential Dumping | `how_dump_credentials.md` |
| Lateral Movement | `lateral_movement.md` |
| DCSync | `DCSync_attack.md` |
| Golden/Silver Tickets | `golden_silver_ticket_attacks.md` |
| BloodHound Enumeration | `bloodhound_AD_enumeration.md` |
| PowerView Enumeration | `powerview_AD_enumeration.md` |
| GPO Abuse | `GPO_abuse_attacks.md` |

---

## 🎯 Suggested Attack Order (Beginner to Advanced)

```text
1. LLMNR Poisoning (Responder) → Capture hashes
2. Password Spraying → Find weak passwords
3. AS-REP Roasting → Target misconfigurations
4. Enumeration (PowerView/BloodHound) → Map the domain
5. Kerberoasting → Crack service accounts
6. Credential Dumping (Mimikatz) → Extract hashes
7. Lateral Movement (Pass-the-Hash) → Move to other machines
8. DCSync → Extract all domain hashes
9. Golden Ticket → Achieve persistence
10. GPO Abuse → Domain-wide control
```

---

## 📚 Free Practice Platforms (Alternative to Home Lab)

| Platform | Description | URL |
| --- | --- | --- |
| Hack The Box | VPN-based AD labs | hackthebox.com |
| TryHackMe | Guided AD rooms | tryhackme.com |
| PentesterLab | Web + AD exercises | pentesterlab.com |
| Vulnerable AD (VulnAD) | Pre-built vulnerable AD | GitHub |
| DVAD | Damn Vulnerable Active Directory | GitHub |
| GOAD | Game of Active Directory | GitHub |

---

## 🎓 Recommended Certifications

| Certification | Focus |
| --- | --- |
| OSCP (OffSec) | General pentesting + AD |
| CRTP (Pentester Academy) | AD attack & defense |
| CRTE (Pentester Academy) | Advanced AD exploitation |
| HTB CPTS | Practical pentesting |
| eCPPT (INE) | Penetration testing |

---

## ⚠ Ethical Reminder

* This lab is for **learning ONLY**
* Never use these techniques on systems without **written authorization**
* Understanding attacks helps you **build better defenses**
* Always follow **responsible disclosure** practices

---

## 📚 What to Learn Next

* 🔥 Start with LLMNR Poisoning (easiest first win)
* 🧠 Progress through the attack order above
* 🛡 After each attack, learn the defensive countermeasures
* 🎓 Work toward OSCP or CRTP certification
