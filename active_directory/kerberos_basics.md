# 🔐 Kerberos Basics

## 📌 What Is Kerberos?

Kerberos is a **network authentication protocol**.

It allows users to:

* Prove their identity
* Access services securely
* Avoid sending passwords over the network

In Active Directory, **Kerberos is the default authentication protocol**.

---

# 🎯 Main Goal of Kerberos

> Authenticate users securely using tickets instead of passwords.

Instead of sending your password repeatedly,
Kerberos gives you a **ticket**.

---

# 🧠 Simple Real-World Analogy

Think of Kerberos like an airport system:

1. You show ID at check-in → get boarding pass.
2. You use boarding pass to enter gate.
3. You don’t show passport every time.

Your boarding pass = Kerberos ticket.

---

# 🏗 Components of Kerberos in AD

There are 3 main parts:

## 1️⃣ Client

The user or computer trying to log in.

## 2️⃣ KDC (Key Distribution Center)

Runs on the Domain Controller.

The KDC has two services:

* Authentication Service (AS)
* Ticket Granting Service (TGS)

## 3️⃣ Service

File server, web server, SQL server, etc.

---

# 🔄 Kerberos Authentication Flow (Step-by-Step)

This is what happens when you log in to a domain computer.

---

## 🟢 Step 1: AS-REQ (Authentication Service Request)

You enter:

```
username + password
```

Client sends request to Domain Controller.

Important:
Your password is NOT sent directly.
A timestamp is encrypted using your password hash.

---

## 🟢 Step 2: AS-REP (Authentication Service Reply)

If password is correct:

The DC sends back:

🎟 TGT (Ticket Granting Ticket)

This ticket:

* Proves you are authenticated
* Is encrypted with KRBTGT account secret
* Usually valid for 10 hours

You now have a TGT.

---

## 🟢 Step 3: TGS-REQ (Ticket Granting Service Request)

When you try to access a service (like a file share):

Example:

```
\\fileserver\shared
```

Your computer sends the TGT to DC and asks:

> “I want access to fileserver.”

---

## 🟢 Step 4: TGS-REP (Ticket Granting Service Reply)

DC checks TGT.

If valid:
It sends back a **Service Ticket**.

This ticket is encrypted using the service account’s password hash.

---

## 🟢 Step 5: Access Service

You send Service Ticket to fileserver.

Fileserver decrypts it.

If valid:
You get access.

No password sent across network.

---

# 🎫 Types of Kerberos Tickets

## 1️⃣ TGT (Ticket Granting Ticket)

* First ticket you receive
* Used to request service tickets
* Encrypted by KRBTGT account

## 2️⃣ Service Ticket (TGS Ticket)

* Used to access specific services
* Encrypted with service account hash

---

# ⏳ Ticket Lifetime

Default (usually):

* TGT: 10 hours
* Renewable: 7 days

If ticket expires → reauthentication required.

---

# 🔐 Why Kerberos Is Secure

* No plaintext passwords sent
* Mutual authentication (client + server verify each other)
* Uses encryption
* Uses timestamps (prevents replay attacks)

---

# ⚠ Common Kerberos Issues

1. Time difference > 5 minutes → authentication fails
2. DNS misconfiguration → cannot find DC
3. SPN issues → service authentication fails
4. NTLM fallback if Kerberos fails

---

# 🆚 Kerberos vs NTLM

| Feature               | Kerberos    | NTLM     |
| --------------------- | ----------- | -------- |
| Ticket-based          | Yes         | No       |
| Mutual authentication | Yes         | No       |
| Secure                | More secure | Older    |
| Default in modern AD  | Yes         | Fallback |

Modern security best practice:
👉 Reduce NTLM usage.

---

# 🧠 Important AD Security Note

The **KRBTGT account** is extremely important.

If compromised:
Attackers can forge Kerberos tickets.

That’s why:

* KRBTGT reset procedure is critical in incidents.

(We study this only for defensive awareness.)

---

# 🔍 Useful Commands (Safe Viewing)

Check your Kerberos tickets:

```cmd
klist
```

Purge tickets:

```cmd
klist purge
```


# 1️⃣ What is a TGT?

## 🎫 TGT = Ticket Granting Ticket

A TGT is the **first ticket** you receive after successful login.

It proves:

> “This user has been authenticated by the Domain Controller.”

---

### 🔎 When do you get it?

Immediately after you log in to a domain-joined machine.

---

### 🔐 What does it do?

The TGT allows you to request access to services **without re-entering your password**.

Instead of sending your password again, you present the TGT to the DC.

---

### 🧠 Simple Analogy

TGT = Master access badge
Service Ticket = Door-specific pass

---

### ⏳ Validity

Default:

* Valid ~10 hours
* Renewable for several days

---

# 2️⃣ What is a Service Ticket?

## 🎟 Service Ticket (TGS Ticket)

A Service Ticket is issued when you request access to a specific service.

Example:

```text
\\fileserver\share
SQL Server
Web Server
```

---

### 🔄 How it works:

1. You send TGT to Domain Controller.
2. You ask for access to a service.
3. DC gives you a Service Ticket.
4. You present Service Ticket to that service.
5. Access granted.

---

### 🔐 Important:

Service Ticket is encrypted using the service account’s password hash.

Only that service can decrypt it.

---

# 3️⃣ What Role Does the Domain Controller Play?

The Domain Controller (DC) runs the **KDC – Key Distribution Center**.

The DC:

1. Authenticates your credentials.
2. Issues the TGT.
3. Issues Service Tickets.
4. Maintains user/group database.
5. Stores password hashes.
6. Enforces policies.

---

### 🧠 Think of DC as:

* Identity authority
* Ticket issuer
* Security controller

If DC is down:

* No authentication
* No ticket issuance
* No new logins

---

# 4️⃣ Why Is Time Sync Important?

Kerberos uses **timestamps** to prevent replay attacks.

When you log in:

* Your request contains a timestamp.
* DC checks if it’s within allowed time window.

---

### ⏳ Default tolerance:

Maximum 5 minutes difference.

If system clock differs more than that:

❌ Authentication fails.

---

### Why?

Prevents attackers from capturing and replaying old authentication messages.

---

### In AD Environment:

The Domain Controller acts as the time source.

All domain-joined machines sync with the DC.

---

# 5️⃣ Why Is KRBTGT Critical?

KRBTGT is a special built-in account in Active Directory.

It is used to:

> Encrypt and sign all TGTs.

---

### 🔐 Why it matters:

When the DC issues a TGT:

* It encrypts it using the KRBTGT account’s secret key.

If KRBTGT secret is compromised:

* Fake TGTs can be generated.
* Authentication trust can be abused.

---

### 🚨 In Security Incidents:

If AD is compromised:
One critical recovery step is:

* Reset KRBTGT password (twice).

This invalidates all existing TGTs.

---

# 🔎 Quick Summary Table

| Concept           | Simple Meaning                     |
| ----------------- | ---------------------------------- |
| TGT               | Proof you logged in successfully   |
| Service Ticket    | Access pass for a specific service |
| Domain Controller | Ticket issuer + identity authority |
| Time Sync         | Prevents replay attacks            |
| KRBTGT            | Secret key used to sign TGTs       |

---

# 🧠 If You Understand This, You Understand:

* Why DC compromise is catastrophic
* Why time sync errors break authentication
* Why KRBTGT reset is major during incident response
* Why Kerberos is stronger than NTLM

