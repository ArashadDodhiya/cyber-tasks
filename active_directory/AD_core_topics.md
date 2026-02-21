# 1️⃣ What Is Active Directory?

**Active Directory (AD)** is Microsoft’s centralized identity and access management system.

It allows organizations to:

* Authenticate users (Who are you?)
* Authorize access (What can you access?)
* Manage computers
* Apply security policies
* Centralize administration

Think of AD as:

> 🏢 The company’s digital identity database + rule engine.

It stores:

* Users
* Password hashes
* Groups
* Computers
* Policies
* Permissions

---

# 2️⃣ Domain vs Workgroup

This is critical.

## 🖥 Workgroup (Small Setup)

* Each computer manages its own users.
* No central authentication.
* Used in home/small office environments.
* No centralized policy.

Example:
PC1 has user “John”
PC2 has its own “John” (completely separate)

No trust between them.

---

## 🏢 Domain (Enterprise Setup)

* Central authentication via Domain Controller
* One username works across all machines
* Central policy enforcement
* Centralized user management

Example:
User logs into any domain-joined machine with:

```
john@company.local
```

Credentials are verified by a Domain Controller.

---

### 🔐 Security Difference

| Workgroup      | Domain                  |
| -------------- | ----------------------- |
| Local accounts | Centralized accounts    |
| No Kerberos    | Kerberos authentication |
| No GPO         | Group Policy            |
| Hard to manage | Scalable                |

Domains are secure and scalable. Workgroups are not enterprise-ready.

---

# 3️⃣ Forest vs Domain

This confuses many beginners.

## 🌳 Domain

A domain is:

* A logical boundary
* Shares a common database
* Shares authentication

Example:

```
company.local
```

---

## 🌲 Forest

A forest is:

* A collection of one or more domains
* Shares trust relationships
* Shares schema

Example:

```
Forest: corp.local
 ├── sales.corp.local
 ├── hr.corp.local
 └── dev.corp.local
```

All domains inside a forest:

* Trust each other (by default)
* Share global catalog
* Share schema

---

### Simple Explanation:

Domain = One company division
Forest = Entire organization structure

---

# 4️⃣ Domain Controller (DC)

The Domain Controller is:

> The brain of Active Directory.

It:

* Stores AD database (NTDS.dit)
* Authenticates users
* Issues Kerberos tickets
* Enforces policies
* Replicates data to other DCs

---

### What happens when you log in?

1. You enter credentials.
2. DC verifies password hash.
3. DC issues Kerberos Ticket Granting Ticket (TGT).
4. You gain access based on group membership.

---

### Important:

If DC goes down:

* No authentication
* No new logins
* Major outage

That’s why enterprises have multiple DCs.

---

# 5️⃣ LDAP Basics

LDAP = Lightweight Directory Access Protocol

It is:

> The protocol used to query and modify Active Directory.

When tools search AD, they use LDAP.

Example LDAP query:

```
Find all users in HR OU
```

Uses:

* TCP 389 (standard)
* TCP 636 (LDAPS – secure)

AD stores data in a hierarchical structure similar to folders.

Example:

```
CN=John Doe,OU=HR,DC=company,DC=local
```

Breakdown:

* CN = Common Name
* OU = Organizational Unit
* DC = Domain Component

LDAP is how applications talk to AD.

---

# 6️⃣ DNS in Active Directory

DNS is **critical** for AD.

Without DNS → AD breaks.

Why?

* Clients locate Domain Controllers using DNS.
* DCs register SRV records.
* Kerberos relies on proper name resolution.

Example DNS record:

```
_ldap._tcp.dc._msdcs.company.local
```

This tells clients where DCs are.

---

### Common AD Issue:

Misconfigured DNS = authentication failures.

Best Practice:

* Domain clients must use internal AD DNS server.
* Never use public DNS (like 8.8.8.8) on domain machines.

---

# 7️⃣ OU Structure (Organizational Units)

OUs are containers inside a domain.

They are used for:

* Organizing objects
* Delegating administration
* Applying Group Policy

Example:

```
company.local
 ├── IT
 │    ├── Users
 │    └── Computers
 ├── HR
 └── Servers
```

---

### Important Concept:

OUs are **not security boundaries**.

They are administrative containers.

Security boundaries:

* Forest
* Domain

---

# 8️⃣ Groups (Security vs Distribution)

## 🔐 Security Groups

Used to:

* Assign permissions
* Control access
* Apply GPO filtering

Example:

* File share access
* Local admin rights
* Application access

Security groups affect access tokens during login.

---

## 📧 Distribution Groups

Used for:

* Email distribution lists
* Messaging only

They do NOT:

* Grant permissions
* Affect authentication

---

### Group Scope Types (Important)

You should also know:

* Domain Local
* Global
* Universal

But we’ll go deeper into that next if you want.

---