# 🔎 What Is PowerView?

**PowerView** is a PowerShell-based Active Directory enumeration tool.

It is commonly used to:

* Query AD objects
* Analyze permissions
* Identify misconfigurations
* Map privilege relationships

It was originally part of the **PowerSploit** framework.

⚠ Important:
PowerView is used heavily in red team engagements — but defenders must understand it to detect and prevent abuse.

Only use it in:

* Lab environments
* Authorized penetration tests
* Security research

---

# 🧠 Why Learn PowerView?

Because it shows you:

* What attackers can see from a normal domain user account
* How privilege escalation paths are discovered
* Where misconfigurations exist

Understanding it makes you a better defender.

---

# 🛠 What PowerView Does (In Simple Terms)

It allows you to:

* List users
* List groups
* Check group membership
* Find privileged users
* Identify delegation
* Review ACL permissions
* Map trust relationships

It basically queries AD using LDAP in a more flexible way than default cmdlets.

---

# 📥 Loading PowerView (Lab Only)

In a lab:

```powershell
Import-Module .\PowerView.ps1
```

Or:

```powershell
. .\PowerView.ps1
```

(That dot + space is important in PowerShell.)

---

# 🔍 Basic PowerView Enumeration Commands

---

## 👤 Get All Users

```powershell
Get-DomainUser
```

---

## 👥 Get All Groups

```powershell
Get-DomainGroup
```

---

## 👑 Check Domain Admins

```powershell
Get-DomainGroupMember -Identity "Domain Admins"
```

---

## 🖥 Get Domain Computers

```powershell
Get-DomainComputer
```

---

## 🌐 Get Domain Controllers

```powershell
Get-DomainController
```

---

# 🔐 Security-Focused Enumeration

These are what security professionals analyze.

---

## 🔎 Find Users with SPNs (Service Accounts)

```powershell
Get-DomainUser -SPN
```

Why important?
Service accounts are often misconfigured or overprivileged.

---

## 🔍 Find AdminCount = 1 Accounts

```powershell
Get-DomainUser -AdminCount
```

These are protected / previously privileged accounts.

---

## 🔑 Find Delegation Settings

Unconstrained delegation:

```powershell
Get-DomainComputer -Unconstrained
```

Constrained delegation:

```powershell
Get-DomainUser -TrustedToAuth
```

Delegation misconfigurations are common attack paths.

---

## 📜 Review ACL Permissions

Check object permissions:

```powershell
Get-DomainObjectAcl -Identity "OU=IT,DC=company,DC=local"
```

Look for risky rights like:

* GenericAll
* WriteDACL
* WriteOwner

This is how privilege escalation paths are discovered.

---

# 🧠 What Makes PowerView Powerful?

Built-in AD module:

* Designed for administration.

PowerView:

* Designed for flexible enumeration.
* Easier privilege path discovery.
* Better filtering of security-relevant data.

---

# 🔴 Why Defenders Must Understand PowerView

Because attackers:

1. Compromise 1 low-priv user.
2. Use tools like PowerView.
3. Map privilege escalation paths.
4. Move toward Domain Admin.

If you understand what PowerView can see,
you know what to harden.

---

# 🛡 How to Defend Against PowerView Usage

* Enable PowerShell logging
* Enable Script Block Logging
* Monitor Event ID 4104
* Monitor LDAP query spikes
* Monitor abnormal AD enumeration behavior
* Restrict unnecessary read permissions (where possible)

---

# 🔥 Modern Reality

Many environments now detect:

* PowerView signatures
* Suspicious LDAP enumeration
* Abnormal Kerberos behavior

But misconfigured AD is still extremely common.

---

# 🧪 Recommended Safe Practice

Build a lab:

1. Create:

   * 3 OUs
   * 20 users
   * Nested groups
2. Misconfigure:

   * Add excessive delegation
   * Add service accounts with high privilege
3. Use PowerView to:

   * Discover misconfigs
   * Map privilege chains

Then fix them.

That’s how you truly learn AD security.

---

# 📈 Where This Leads

If you master PowerView, next topics should be:

* BloodHound (graph-based privilege mapping)
* AD ACL deep dive
* Delegation abuse concepts (defensive view)
* AdminSDHolder
* Attack path reduction strategies

---

If you tell me your goal:

* 🔴 Red team path (ethical & lab)
* 🔵 Blue team detection focus
* 🟣 AD security architect path

I’ll tailor the next level properly.
