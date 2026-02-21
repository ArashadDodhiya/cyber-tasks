# 🌐 DNS – Full Form & Explanation

## ✅ Full Form:

**DNS = Domain Name System**

---

## 📌 What Is DNS?

DNS is the system that translates **human-readable names** into **IP addresses**.

Example:

Instead of remembering:

```
192.168.1.10
```

You use:

```
server.company.local
```

DNS converts the name → IP address.

---

## 🧠 Simple Analogy

DNS is like a **phonebook for computers**.

You search:

> “Domain Controller”

DNS responds:

> “Its IP address is 192.168.1.10”

---

## 🔐 Why DNS Is Critical in Active Directory

In AD, DNS does more than just name resolution.

It helps clients:

* Find Domain Controllers
* Find Global Catalog servers
* Locate LDAP services
* Locate Kerberos services

Example of special AD DNS record:

```
_ldap._tcp.dc._msdcs.company.local
```

This is called an **SRV record**.

It tells clients:

> “Here are the Domain Controllers available.”

---

## ⚠ If DNS Breaks in AD:

* Users cannot log in
* GPOs won’t apply
* Kerberos fails
* Domain join fails

💡 Important Rule:
Domain-joined machines must use **internal AD DNS server**, not public DNS like 8.8.8.8.

---

# 🛡 GPO – Full Form & Explanation

## ✅ Full Form:

**GPO = Group Policy Object**

---

## 📌 What Is a GPO?

A GPO is a collection of settings that control:

* Security settings
* Password policies
* Software deployment
* Desktop configuration
* Scripts
* Windows updates
* Administrative templates

It allows centralized control of domain machines and users.

---

## 🧠 Simple Analogy

Think of GPO as:

> The company rulebook automatically applied to all employees' computers.

Instead of configuring 500 PCs manually,
you create one GPO and apply it to an OU.

---

## 🏗 How GPO Works

1. GPO is created in:

   ```
   gpmc.msc
   ```
2. It is linked to:

   * Site
   * Domain
   * OU
3. It applies based on:

   * LSDOU order (Local → Site → Domain → OU)
4. Client pulls policy from Domain Controller.

---

## 🔐 Types of GPO Settings

### Computer Configuration

Applies to the machine.

Example:

* Disable USB
* Firewall settings
* BitLocker policy

---

### User Configuration

Applies to the user.

Example:

* Disable Control Panel
* Map network drives
* Set desktop wallpaper

---

## ⚠ Why GPO Is Security-Critical

Misconfigured GPO can:

* Give users local admin rights
* Expose file shares
* Run insecure startup scripts
* Weaken password policies

Well-configured GPO can:

* Enforce password complexity
* Enable auditing
* Enforce firewall rules
* Disable legacy protocols

---

# 🔍 Quick Comparison

| Term | Full Form           | Purpose                                       |
| ---- | ------------------- | --------------------------------------------- |
| DNS  | Domain Name System  | Resolves names to IP addresses                |
| GPO  | Group Policy Object | Enforces centralized configuration & security |

---