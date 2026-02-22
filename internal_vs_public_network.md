# 🏢 What Does “Internal” Mean in Networking?

**Internal** means:

> Systems that are inside a private network and **not directly accessible from the internet**.

These machines usually:

* Have **private IP addresses**
* Are protected by firewalls
* Are meant to be accessed only by employees or internal systems

---

# 🌍 Public vs Internal (Simple Comparison)

| Public                   | Internal                       |
| ------------------------ | ------------------------------ |
| Accessible from internet | Only accessible inside network |
| Has public IP            | Has private IP                 |
| Example: company website | Example: company database      |

---

# 🧠 Real-World Example

Imagine a company called **TechCorp**.

### From the Internet, You Can See:

```text
https://techcorp.com
Web Server: 203.0.113.10
```

This server is public.

---

### Inside the Company Network

There are other systems:

```text
Domain Controller: 10.0.0.10
Database Server: 10.0.0.20
HR System: 10.0.0.30
File Server: 10.0.0.40
```

These use **private IP addresses**.

Private IP ranges:

* 10.0.0.0 – 10.255.255.255
* 172.16.0.0 – 172.31.255.255
* 192.168.0.0 – 192.168.255.255

These cannot be accessed directly from the internet.

---

# 📊 Visual Network Diagram

```
                 INTERNET
                     |
                     |
              [ Public Web Server ]
              203.0.113.10
                     |
              (Firewall / NAT)
                     |
              -----------------
              |               |
      10.0.0.10        10.0.0.20
  Domain Controller    Database
              |
         10.0.0.40
         File Server
```

The bottom systems are **internal**.

---

# 🧨 Why Attackers Care About Internal Access

The public web server usually has:

* Limited privileges
* Minimal data
* Hardened security

But the internal systems contain:

* Password hashes
* Employee data
* Financial records
* Backup systems
* Active Directory

That’s where real value is.

---

# 🔥 Example Attack Scenario

1. Attacker exploits web server vulnerability.

2. Gains shell access on:

   ```
   203.0.113.10
   ```

3. From that server, they scan:

   ```bash
   nmap 10.0.0.0/24
   ```

4. They discover:

   ```
   10.0.0.10 - Active Directory
   10.0.0.20 - SQL Database
   ```

These systems were **never visible from the internet**.

This is what "internal access" means.

---

# 🏠 Think of It Like a House

Public server = Front door
Internal systems = Bedrooms, safe, vault

Breaking into the porch is one thing.
Getting into the safe room is another.

---

# 🧠 Corporate Example

In environments using Microsoft Active Directory:

Internal systems might include:

* Domain Controllers
* Exchange servers
* Backup servers
* HR databases

None of these are public-facing.

---

# 🔐 Why Internal Is Usually More Dangerous

Because internal networks often:

* Trust each other
* Have fewer restrictions
* Allow SMB, RDP, LDAP
* Assume "if you're inside, you're trusted"

That assumption is often wrong.

---

# 🧪 Example with SSH Pivoting

You compromise:

```
Web Server (Public)
203.0.113.10
```

It can reach:

```
10.0.0.20:3306 (MySQL)
```

You use:

```bash
ssh -L 3306:10.0.0.20:3306 user@203.0.113.10
```

Now you can access the internal database.

Without that foothold — you couldn't.

---

# 🎯 Summary

Internal = Private network systems that:

* Are behind firewall
* Use private IP addresses
* Are not internet-exposed
* Contain critical company resources

Most major breaches happen **after** attackers gain internal access.

---

* 🔬 What internal network segmentation looks like
* 🧠 How attackers move laterally inside
* 🛡 How companies protect internal networks
* 📊 A full step-by-step attack story
