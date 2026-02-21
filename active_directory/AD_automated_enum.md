# 🔎 What Is Automated AD Enumeration?

**Automated enumeration** means:

> Using scripts or tools to automatically collect information about Active Directory instead of manually checking everything.

Instead of:

* Opening ADUC
* Clicking groups one by one
* Manually reviewing permissions

You run:

* A script
* A tool
* A query

And it collects everything in minutes.

---

# 🎯 Why Automation Matters

In real environments:

* 10,000+ users
* 5,000+ computers
* Hundreds of groups
* Nested memberships
* Delegation
* ACLs

Manual review is impossible.

Automation helps you:

* Detect misconfigurations
* Identify risky permissions
* Map privilege paths
* Audit compliance
* Prepare for pentesting (authorized)

---

# 🧰 Types of Automated AD Enumeration

There are 4 main approaches:

---

# 1️⃣ Built-in PowerShell (Safe Admin Method)

Best for administrators.

## Example: Enumerate All Users

```powershell
Get-ADUser -Filter * | Select Name,Enabled
```

---

## Find Privileged Users

```powershell
Get-ADGroupMember "Domain Admins"
```

---

## Find Users with SPNs

```powershell
Get-ADUser -Filter {ServicePrincipalName -like "*"} -Properties ServicePrincipalName
```

---

## Find Disabled Accounts

```powershell
Search-ADAccount -AccountDisabled -UsersOnly
```

---

## Export Everything to CSV

```powershell
Get-ADUser -Filter * | Export-Csv users.csv -NoTypeInformation
```

This is the **clean enterprise way**.

---

# 2️⃣ Script-Based Enumeration

You create a script that:

* Collects users
* Collects groups
* Collects nested memberships
* Checks adminCount
* Checks delegation
* Exports findings

Example structure:

```powershell
$users = Get-ADUser -Filter * -Properties adminCount,PasswordNeverExpires
$groups = Get-ADGroup -Filter *
$da = Get-ADGroupMember "Domain Admins"

$users | Export-Csv users.csv -NoTypeInformation
$groups | Export-Csv groups.csv -NoTypeInformation
$da | Export-Csv domain_admins.csv -NoTypeInformation
```

Now enumeration is automated in seconds.

---

# 3️⃣ Advanced Enumeration Tools (Lab / Security Research)

These tools automate deep analysis.

## 🔎 PowerView

* LDAP-based enumeration
* ACL analysis
* Delegation discovery

## 🧠 BloodHound

* Maps privilege relationships as a graph
* Shows attack paths visually
* Excellent for defenders to identify risky chains

## 📊 PingCastle

* AD security health check
* Risk scoring
* Best practice evaluation

## 🔍 ADExplorer

* Snapshot of entire AD database
* Offline analysis

⚠ Only use in:

* Lab
* Authorized assessments

---

# 4️⃣ LDAP Query Automation

Since AD runs on LDAP, you can automate:

* User searches
* Permission analysis
* Object discovery

Example:

```powershell
Get-ADObject -LDAPFilter "(adminCount=1)"
```

LDAP filters are powerful for large-scale enumeration.

---

# 🛡 What Automated Enumeration Typically Looks For

Security-focused enumeration checks:

### 🔐 Privilege Risks

* Members of Domain Admins
* Enterprise Admins
* Administrators
* Backup Operators
* Account Operators

---

### 🔐 Account Risks

* Password never expires
* Inactive accounts
* Disabled but not removed accounts
* Service accounts with high privilege

---

### 🔐 Delegation Risks

* Unconstrained delegation
* Constrained delegation
* Resource-based delegation

---

### 🔐 ACL Risks

* GenericAll permissions
* WriteDACL
* WriteOwner
* Over-permissive OU rights

---

### 🔐 Infrastructure Risks

* NTLM enabled everywhere
* LDAP signing disabled
* SMB signing disabled
* Weak password policy

---

# 🚨 Why Defenders Must Understand Automation

Because attackers:

1. Compromise one user
2. Run automated enumeration
3. Map privilege paths
4. Escalate quickly

Automation reduces discovery time from days → minutes.

If you can automate audits, you can close gaps early.

---

# 📊 Blue Team Detection Focus

Monitor:

* Unusual LDAP queries
* PowerShell logging (Event ID 4104)
* Excessive group membership queries
* Abnormal Kerberos ticket requests
* BloodHound collection behavior

Modern EDR solutions detect heavy AD enumeration.

---

# 🧪 Lab Exercise for You

In your lab:

1. Create:

   * 50 users
   * Nested groups
   * One misconfigured delegation
2. Write a PowerShell script that:

   * Finds privileged users
   * Finds adminCount accounts
   * Finds SPN accounts
   * Exports everything
3. Fix issues and re-run script.

That’s real learning.

---

# 📈 If You Want To Go Deeper

Next research areas:

* Automated attack path reduction
* BloodHound path analysis
* AD ACL graph theory
* Privilege escalation chain modeling
* Continuous AD security monitoring
