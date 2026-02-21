# 🧰 1️⃣ What Is AD PowerShell?

AD PowerShell means using **PowerShell commands (cmdlets)** to manage Active Directory instead of GUI tools like `dsa.msc`.

It allows you to:

* Create users in bulk
* Modify groups
* Audit permissions
* Search AD quickly
* Automate tasks

---

# 📦 2️⃣ Install / Enable AD PowerShell Module

On Domain Controller:
It is usually already installed.

On Windows Client:
Install RSAT (Remote Server Administration Tools).

Then import module:

```powershell
Import-Module ActiveDirectory
```

Check if loaded:

```powershell
Get-Module ActiveDirectory
```

---

# 👤 3️⃣ User Management (Most Important)

---

## 🔍 Get User Information

Get specific user:

```powershell
Get-ADUser john.doe
```

Get full properties:

```powershell
Get-ADUser john.doe -Properties *
```

Get all users:

```powershell
Get-ADUser -Filter *
```

---

## ➕ Create New User

```powershell
New-ADUser -Name "John Doe" `
-SamAccountName jdoe `
-UserPrincipalName jdoe@company.local `
-Path "OU=HR,DC=company,DC=local" `
-AccountPassword (Read-Host -AsSecureString "Enter Password") `
-Enabled $true
```

---

## 🔑 Reset Password

```powershell
Set-ADAccountPassword jdoe -Reset -NewPassword (Read-Host -AsSecureString)
```

---

## 🔒 Disable / Enable User

```powershell
Disable-ADAccount jdoe
Enable-ADAccount jdoe
```

---

# 👥 4️⃣ Group Management

---

## View Groups

```powershell
Get-ADGroup -Filter *
```

---

## Check Group Members

```powershell
Get-ADGroupMember "Domain Admins"
```

---

## Add User to Group

```powershell
Add-ADGroupMember -Identity "IT Support" -Members jdoe
```

---

## Remove User from Group

```powershell
Remove-ADGroupMember -Identity "IT Support" -Members jdoe
```

---

# 💻 5️⃣ Computer Management

---

## List Domain Computers

```powershell
Get-ADComputer -Filter *
```

---

## Get Computer Details

```powershell
Get-ADComputer PC01 -Properties *
```

---

## Disable Computer Account

```powershell
Disable-ADAccount PC01$
```

---

# 🏢 6️⃣ OU (Organizational Unit) Management

---

## List OUs

```powershell
Get-ADOrganizationalUnit -Filter *
```

---

## Create New OU

```powershell
New-ADOrganizationalUnit -Name "IT" -Path "DC=company,DC=local"
```

---

# 🔐 7️⃣ Password & Policy Info

---

## View Default Domain Password Policy

```powershell
Get-ADDefaultDomainPasswordPolicy
```

---

## View Fine-Grained Password Policies

```powershell
Get-ADFineGrainedPasswordPolicy -Filter *
```

---

# 🔍 8️⃣ Searching AD (Very Powerful)

Find disabled users:

```powershell
Search-ADAccount -AccountDisabled -UsersOnly
```

Find locked accounts:

```powershell
Search-ADAccount -LockedOut
```

Find accounts with password never expires:

```powershell
Get-ADUser -Filter {PasswordNeverExpires -eq $true}
```

Find inactive users (90 days):

```powershell
Search-ADAccount -UsersOnly -AccountInactive -TimeSpan 90.00:00:00
```

---

# 🌐 9️⃣ Domain & DC Information

---

## Get Domain Info

```powershell
Get-ADDomain
```

---

## Get Domain Controllers

```powershell
Get-ADDomainController -Filter *
```

---

## Get Forest Info

```powershell
Get-ADForest
```

---

# 🔎 🔟 Security-Focused Checks

Check members of Domain Admins:

```powershell
Get-ADGroupMember "Domain Admins"
```

Find service accounts (SPN):

```powershell
Get-ADUser -Filter {ServicePrincipalName -like "*"} -Properties ServicePrincipalName
```

Check users with adminCount = 1:

```powershell
Get-ADUser -Filter {adminCount -eq 1} -Properties adminCount
```

---

# 📂 1️⃣1️⃣ Exporting Data (Very Useful)

Export all users to CSV:

```powershell
Get-ADUser -Filter * | Export-Csv users.csv -NoTypeInformation
```

Export Domain Admins list:

```powershell
Get-ADGroupMember "Domain Admins" | Export-Csv DA.csv -NoTypeInformation
```

---

# 🔄 1️⃣2️⃣ Bulk Operations (Automation Power)

Example: Add many users from CSV

```powershell
Import-Csv users.csv | ForEach-Object {
    New-ADUser -Name $_.Name -SamAccountName $_.Username -Enabled $true
}
```

This is why PowerShell is powerful.

---

# 🧠 1️⃣3️⃣ Important Concepts You Must Know

## -Filter

Used to search objects.

## -Properties *

Gets all attributes.

## Distinguished Name (DN)

Example:

```
CN=John Doe,OU=HR,DC=company,DC=local
```

---

# 🔥 1️⃣4️⃣ Why PowerShell Is Better Than GUI

GUI:

* Manual
* Slow
* Not scalable

PowerShell:

* Automated
* Scriptable
* Fast
* Auditable
* Used in enterprise environments

---

# 🛡 1️⃣5️⃣ Security Awareness

As ethical professional:

* Always log changes
* Never mass-modify without testing
* Use least privilege
* Avoid running as Domain Admin unnecessarily

PowerShell actions are logged (Event ID 4688 + PowerShell logs).

---

# 📊 Beginner → Advanced Path in AD PowerShell

Level 1:

* Get-ADUser
* Add-ADGroupMember
* Reset passwords

Level 2:

* Bulk automation
* Security auditing
* Filtering queries

Level 3:

* ACL analysis
* Delegation review
* Attack path analysis (defensive)

---

# 🎯 What You Should Practice in Lab

1. Create 10 users using PowerShell.
2. Create 2 OUs.
3. Move users between OUs.
4. Add group memberships.
5. Export inactive accounts.
6. Audit privileged groups.

---

If you want next, I can give you:

* 🧠 AD PowerShell cheat sheet (one-page reference)
* 🛡 Security auditing script examples
* 🔥 Advanced AD PowerShell topics
* 🏗 30-day AD PowerShell mastery plan

Tell me your next step 👇
