# 1️⃣ What is Active Directory?

**Active Directory (AD)** is a **Microsoft directory service** used in Windows domain networks to:

* Manage users
* Manage computers
* Control access to resources
* Apply security policies
* Centralize authentication & authorization

Think of AD as:

> 🏢 The central security system of an organization’s IT environment.

Almost every enterprise network uses Active Directory.

---

# 2️⃣ Why Active Directory is Important in Ethical Hacking

In real-world pentests:

* 90%+ enterprise environments use AD
* Once you compromise AD, you often control the entire network
* Most major breaches involve AD misconfigurations

As an ethical hacker, your goal is to:

* Identify weaknesses
* Simulate real-world attacks
* Help organizations secure AD

---

# 3️⃣ Core Components of Active Directory

Let’s break it down.

---

## 🔹 1. Domain

A **Domain** is a logical group of users and computers.

Example:

```
company.local
```

All users and machines in that domain trust the Domain Controller.

---

## 🔹 2. Domain Controller (DC)

The **Domain Controller** is the most important server in AD.

It:

* Authenticates users
* Stores password hashes
* Enforces security policies
* Handles Kerberos tickets

🔥 If an attacker compromises the DC → full domain compromise.

---

## 🔹 3. Forest

A **Forest** is a collection of domains that share trust relationships.

Example:

```
corp.local
dev.corp.local
hr.corp.local
```

All these may belong to the same forest.

---

## 🔹 4. Organizational Units (OUs)

Used to:

* Group users or computers
* Apply Group Policies
* Delegate administration

Example:

```
OU=HR
OU=IT
OU=Finance
```

---

## 🔹 5. Group Policy (GPO)

GPOs define:

* Password policies
* Software restrictions
* Firewall settings
* Login scripts

Misconfigured GPOs are a major attack vector.

---

# 4️⃣ How Authentication Works in AD

Two main authentication protocols:

---

## 🔐 1. Kerberos (Most Important)

Modern Windows networks use **Kerberos**.

How it works (simplified):

1. User logs in
2. Gets Ticket Granting Ticket (TGT)
3. Uses TGT to request service tickets
4. Accesses services

⚠️ Kerberos is heavily abused in attacks.

---

## 🔐 2. NTLM

Older protocol.
Still used in many environments.
Less secure than Kerberos.

---

# 5️⃣ Important AD Objects

AD contains objects like:

* Users
* Groups
* Computers
* Service Accounts
* Domain Controllers
* Trust relationships

Each object has:

* SID (Security Identifier)
* Attributes
* Permissions

---

# 6️⃣ Active Directory Attack Surface (Ethical Hacking Focus)

Now we get into the security part.

These are common AD attack techniques (high-level overview only):

---

## 🔥 1. Password Attacks

* Password spraying
* Bruteforce (against weak accounts)
* Kerberoasting
* AS-REP Roasting

---

## 🔥 2. Kerberos Attacks

### 🔹 Kerberoasting

* Extract service account ticket
* Crack offline
* Get service account password

### 🔹 Golden Ticket

* Forge Kerberos ticket using KRBTGT hash
* Domain persistence

### 🔹 Silver Ticket

* Forge service ticket

---

## 🔥 3. NTLM Attacks

* NTLM Relay
* Pass-the-Hash
* SMB Relay

---

## 🔥 4. Privilege Escalation in AD

Common paths:

* Misconfigured permissions
* Unquoted service paths
* Writable GPO
* Weak ACLs

Tools like:

* BloodHound
* PowerView
* SharpHound

Used to map privilege escalation paths.

---

## 🔥 5. Lateral Movement

Once inside:

* Pass-the-Hash
* Pass-the-Ticket
* WMI
* PsExec
* SMB
* RDP

---

## 🔥 6. Domain Persistence Techniques

* Golden tickets
* DCShadow
* AdminSDHolder abuse
* Backdoored GPOs

---

# 7️⃣ Common AD Enumeration Tools (Ethical Use Only)

⚠️ Use only in labs or authorized pentests.

### 🛠️ Enumeration

* BloodHound
* SharpHound
* PowerView
* CrackMapExec
* LDAP queries
* net commands

### 🛠️ Credential Attacks

* Mimikatz (credential dumping)
* Rubeus (Kerberos manipulation)
* Impacket tools

---

# 8️⃣ Defensive Side (Blue Team Perspective)

Understanding attacks helps defend.

Key defenses:

* Enforce strong password policy
* Disable NTLM where possible
* Monitor Kerberos anomalies
* Use tiered admin model
* Implement LAPS
* Monitor event logs (4624, 4768, 4769, etc.)
* Use Privileged Access Workstations (PAW)

---

# 9️⃣ AD Security Concepts Every Ethical Hacker Must Know

You should understand:

* SID & RID
* ACL & ACE
* DACL
* Token privileges
* AdminSDHolder
* Trust relationships
* Kerberos ticket structure
* SPN (Service Principal Names)
* LDAP structure
* NTDS.dit database
* SYSVOL share

---

# 🔟 AD Lab Setup for Practice

If you're serious about AD hacking:

### 🖥️ Minimum Lab:

* 1 Domain Controller (Windows Server)
* 1 Windows Client
* 1 Attacker Machine (Kali Linux)

Use:

* VirtualBox or VMware
* Evaluation Windows ISOs

Practice platforms:

* Hack The Box (AD labs)
* TryHackMe (AD rooms)
* PentesterLab
* CRTP / CRTO tracks

---

# 1️⃣1️⃣ Real-World AD Attack Lifecycle

Typical red team chain:

1. Initial access (phishing, exposed service)
2. Local enumeration
3. Credential dumping
4. Lateral movement
5. Privilege escalation
6. Domain admin compromise
7. Persistence
8. Data exfiltration

Almost every major ransomware attack follows this pattern.

---

# 1️⃣2️⃣ Why AD is Difficult to Secure

Because:

* It’s complex
* Legacy protocols exist
* Backward compatibility
* Human misconfigurations
* Poor privilege management

---

# 1️⃣3️⃣ Career Relevance

If you master AD:

* You can work in Red Team
* Blue Team
* SOC
* Incident Response
* Enterprise Security
* Threat Hunting

AD is foundational knowledge.

---

# 1️⃣4️⃣ Advanced Topics (For Future Learning)

* AD Certificate Services (AD CS) attacks
* Shadow Credentials
* Resource-Based Constrained Delegation
* Forest trust attacks
* SID history abuse
* DCShadow
* Hybrid AD (Azure AD / Entra ID)

---

# 🎯 Final Summary

Active Directory is:

* The backbone of enterprise authentication
* The primary attack target in internal pentests
* A goldmine for privilege escalation
* A must-learn skill for ethical hackers

---

If you'd like, I can next:

* 🔍 Create an AD attack roadmap for beginners
* 🛠️ Show a safe AD lab setup guide step-by-step
* 📊 Explain Kerberos deeply
* 🧠 Teach BloodHound from scratch
* 🔐 Explain AD CS attacks

What level are you currently at — beginner, intermediate, or advanced?
