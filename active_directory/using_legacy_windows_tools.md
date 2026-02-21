# 1️⃣ What Is Active Directory (Quick Recap)

Active Directory (AD) is Microsoft’s directory service used to:

* Authenticate users
* Authorize access to resources
* Manage computers and users in domains
* Enforce Group Policies
* Centralize identity management

Key components:

* **Domain Controller (DC)**
* **Users & Groups**
* **Organizational Units (OUs)**
* **Group Policy Objects (GPOs)**
* **Kerberos authentication**

---

# 2️⃣ Legacy Windows Tools for Active Directory

These are built-in or classic Microsoft tools (often used in older or still-running enterprise environments).

---

## 🖥 1. `dsa.msc` – Active Directory Users and Computers (ADUC)

**Command:**

```
dsa.msc
```

Used for:

* Creating users
* Resetting passwords
* Managing groups
* Moving objects between OUs
* Viewing group memberships

🔐 Ethical Hacker Use:

* Check for:

  * Users in Domain Admins
  * Password policies
  * Service accounts
  * Stale accounts
  * Delegated permissions

---

## 🖥 2. `gpmc.msc` – Group Policy Management Console

**Command:**

```
gpmc.msc
```

Used for:

* Managing Group Policy Objects
* Linking GPOs to OUs
* Reviewing security settings

🔐 Security Focus:

* Weak password policy?
* Misconfigured startup scripts?
* Insecure file shares?
* Overly permissive security filtering?

---

## 🖥 3. `net` Command (Very Important Legacy Tool)

Classic command-line enumeration tool.

### View Domain Users

```
net user /domain
```

### View Domain Admins

```
net group "Domain Admins" /domain
```

### View All Groups

```
net group /domain
```

🔐 Ethical Hacker Use:

* Initial internal enumeration
* Privilege mapping
* Quick recon without installing tools

---

## 🖥 4. `nltest`

Used for domain controller and trust enumeration.

### Check Domain Controllers

```
nltest /dclist:domain.local
```

### Verify Secure Channel

```
nltest /sc_verify:domain.local
```

🔐 Use:

* Validate domain connectivity
* Identify DCs
* Check trust relationships

---

## 🖥 5. `dsquery` (Very Useful Legacy Tool)

Example:

### List Users

```
dsquery user
```

### List Domain Controllers

```
dsquery server
```

### List Computers

```
dsquery computer
```

🔐 Useful for:

* Structured AD enumeration
* Target identification
* Finding disabled accounts

---

## 🖥 6. `setspn`

Used to view Service Principal Names.

### List SPNs for a User

```
setspn -L username
```

🔐 Security Relevance:

* Detect service accounts
* Identify Kerberos attack surfaces (for example, in red team simulations)
* Misconfigured SPNs

---

## 🖥 7. `gpresult`

Used to check applied policies.

```
gpresult /r
```

Or for detailed output:

```
gpresult /h report.html
```

🔐 Use:

* Verify policy application
* Detect misapplied GPOs
* Troubleshoot security baselines

---

## 🖥 8. `dcdiag`

Domain Controller diagnostic tool.

```
dcdiag
```

Checks:

* Replication
* DNS issues
* SYSVOL health

🔐 Blue Team Use:

* Identify misconfiguration
* Detect broken security infrastructure

---

# 3️⃣ Ethical Hacker Perspective (Internal Assessment)

If you’re performing a **legal internal penetration test**, legacy tools are valuable because:

✔ They don’t trigger AV easily
✔ They’re built-in (Living Off the Land)
✔ They blend with admin activity

Common safe assessment steps:

1. Identify domain
2. Enumerate users and groups
3. Identify privileged accounts
4. Review password policies
5. Check service accounts
6. Look for misconfigured GPOs
7. Validate DC health

⚠ Always ensure you have written authorization before testing.

---

# 4️⃣ Practice Lab Setup (Safe Learning)

You can safely practice by:

* Installing:

  * Windows Server (Domain Controller)
  * 1–2 Windows clients
* Creating:

  * Users
  * OUs
  * GPOs
  * Service accounts

Tools to build lab:

* VirtualBox
* VMware
* Hyper-V

---

# 5️⃣ Modern Trend Awareness

Even though these are “legacy” tools:

* Many enterprises still use them.
* Modern red teams still leverage `net`, `nltest`, `dsquery`.
* Attack detection often depends on monitoring these commands.
* Blue teams are improving logging via:

  * PowerShell logging
  * Advanced audit policies
  * SIEM detection rules

Understanding legacy tools helps both offense and defense.

---

If you'd like, tell me your level:

* 🟢 Beginner (new to AD)
* 🟡 Intermediate (admin knowledge)
* 🔴 Advanced (pentesting / red team focus)

And I’ll tailor a hands-on learning roadmap for you.
