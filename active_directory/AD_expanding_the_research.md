# 🏗 1️⃣ AD Architecture Deep Research

Understand how AD is built internally.

## 🔎 A. Logical Structure

* Forest
* Domain
* Tree
* OU
* Sites
* Trusts

Research:

* Why forest is security boundary
* Transitive vs non-transitive trust
* External vs forest trust

---

## 🔎 B. Physical Structure

* Domain Controllers
* Global Catalog
* Sites & Subnets
* Replication topology

Research:

* How replication works (multi-master)
* NTDS.dit database
* SYSVOL folder
* DRS (Directory Replication Service)

---

## 🔎 C. FSMO Roles (Very Important)

5 FSMO Roles:

Forest level:

* Schema Master
* Domain Naming Master

Domain level:

* RID Master
* PDC Emulator
* Infrastructure Master

Research:

* What happens if PDC Emulator fails?
* Why RID Master matters?
* How to transfer vs seize FSMO roles?

---

# 🔐 2️⃣ Authentication Deep Research

## 🔎 A. Kerberos Internals

* TGT structure
* Service Ticket structure
* PAC (Privilege Attribute Certificate)
* Ticket encryption types
* AES vs RC4
* Ticket lifetime policies

Research:

* How KRBTGT signs TGT
* What happens during renewal
* How delegation works

---

## 🔎 B. NTLM

* NTLMv1 vs NTLMv2
* NTLM fallback scenarios
* NTLM relay risk concept (defensive understanding)

Modern research focus:
👉 How to reduce NTLM in enterprise.

---

# 🛡 3️⃣ AD Security Research

This is where real expertise begins.

---

## 🔎 A. Privilege Model

Research:

* AdminSDHolder
* SDProp process
* adminCount attribute
* Protected groups
* Token size & group nesting

---

## 🔎 B. Delegation Types

* Unconstrained Delegation
* Constrained Delegation
* Resource-Based Constrained Delegation (RBCD)

Understand:

* Why delegation is dangerous if misconfigured
* How to detect risky delegation

---

## 🔎 C. ACL & Permission Abuse Concepts

Research:

* DACL vs SACL
* WriteDACL
* WriteOwner
* GenericAll
* Extended Rights
* Object inheritance

This is the foundation of privilege escalation paths.

---

## 🔎 D. Service Accounts & SPNs

Research:

* Service Principal Names
* gMSA (Group Managed Service Accounts)
* Why service accounts are high risk
* Password rotation best practices

---

# 📊 4️⃣ AD Enumeration & Attack Path Research (Defensive Focus)

Understand what attackers analyze:

* User → Group → Privileged Group mapping
* ACL privilege chains
* Delegation paths
* Trust abuse paths
* SID History misuse

Tools (for lab study only):

* PowerView
* BloodHound
* ADExplorer
* PingCastle

Goal:
👉 Identify attack paths before attackers do.

---

# 🧠 5️⃣ AD Hardening Research

Modern enterprise best practices:

## 🔐 Tiered Model

* Tier 0 → Domain Controllers
* Tier 1 → Servers
* Tier 2 → Workstations

## 🔐 PAW (Privileged Access Workstations)

## 🔐 Just-In-Time access

## 🔐 Just-Enough Administration

## 🔐 Credential Guard

## 🔐 LAPS (Local Admin Password Solution)

## 🔐 Windows Defender for Identity

---

# 🌐 6️⃣ Hybrid AD Research (Modern Direction)

Most companies now run hybrid identity.

Research:

* Azure AD Connect
* Entra ID
* Password Hash Sync
* Pass-through Authentication
* Seamless SSO
* Cloud Kerberos Trust
* Conditional Access

Future AD work = hybrid security.

---

# 🚨 7️⃣ AD Incident Response Research

Advanced level:

Research:

* KRBTGT reset procedure
* Golden ticket containment (conceptual)
* DC compromise recovery
* Forest recovery planning
* Backup strategies
* Replication attack detection
* Event log analysis

Critical Event IDs:

* 4624
* 4672
* 4768 / 4769
* 4728 / 4729
* 4742

---

# 📚 8️⃣ Research Tools & Resources

Microsoft:

* Microsoft Learn (AD DS documentation)
* AD security best practices guide
* TechNet deep dives

Tools for lab:

* ADExplorer
* PingCastle (security assessment)
* BloodHound (privilege graphing)
* RSAT tools
* PowerShell AD module

---

# 🧩 9️⃣ Advanced Topics Few People Study

If you want to go deep:

* SID History abuse concept
* Machine account quota
* DNS zone security
* LDAP signing enforcement
* SMB signing
* AD CS (Certificate Services) misconfigurations
* DCShadow concept (defensive research)
* Replication metadata analysis
* KRBTGT double reset logic

---

# 🧭 Research Strategy Recommendation

Instead of random reading:

Phase 1 → Architecture
Phase 2 → Authentication
Phase 3 → Privilege Model
Phase 4 → Delegation & ACL
Phase 5 → Hardening
Phase 6 → Detection & IR

