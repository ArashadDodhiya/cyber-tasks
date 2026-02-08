# 🔴 What is SMTP Enumeration?

**SMTP enumeration** is an **active information-gathering technique** used to:

* Verify **valid email users**
* Identify **mail server behavior**
* Gather info for **phishing, brute force, or lateral movement**

It abuses **SMTP commands** like `VRFY`, `EXPN`, and `RCPT TO`.

👉 This is **ACTIVE recon** because you directly interact with the mail server.

---

# 📮 Quick SMTP Basics (needed to understand enumeration)

* **SMTP port**: `25` (default), `587` (submission), `465` (SMTPS)
* SMTP servers accept **commands in plain text**
* If misconfigured, they **leak user existence**

---

# 1️⃣ `nc -rw 25 <smtp_server>`

### Correct command

```bash
nc -rw 25 <smtp_server_ip>
```

### Example

```bash
nc -rw 25 192.168.50.10
```

### Meaning

* `nc` → netcat (network utility)
* `-r` → randomize ports (optional, legacy)
* `-w` → timeout
* `25` → SMTP port

### What happens

You manually connect to the SMTP service.

Expected banner:

```text
220 mail.example.com ESMTP Postfix
```

📌 Banner itself leaks:

* Mail server software
* Sometimes OS or hostname

---

# 2️⃣ SMTP Enumeration using `VRFY`

### Command (typed **after connecting**)

```text
VRFY root
```

or

```text
VRFY admin
```

### Meaning

`VRFY` asks the mail server:

> “Does this user exist?”

---

### Possible responses

✅ **User exists**

```text
252 Cannot VRFY user, but will accept message
```

or

```text
250 root
```

❌ **User does NOT exist**

```text
550 User unknown
```

🔥 This allows **username enumeration**

---

# 3️⃣ Why `VRFY` is dangerous

Attackers can:

* Build valid email lists
* Launch phishing attacks
* Attempt password attacks
* Target internal users

That’s why modern servers **disable `VRFY`**.

---

# 4️⃣ SMTP Enumeration on Windows

### Enable Telnet Client (one-time)

Run in **Administrator CMD or PowerShell**:

```powershell
dism /online /Enable-Feature /FeatureName:TelnetClient
```

This installs the Telnet client.

---

### Connect to SMTP server using Telnet

```bash
telnet <smtp_server_ip> 25
```

Example:

```bash
telnet 192.168.50.10 25
```

You’ll see:

```text
220 mail.example.com Microsoft ESMTP MAIL Service
```

---

### Perform enumeration

```text
VRFY administrator
VRFY john
```

Same logic as `nc`.

---

# 5️⃣ Other SMTP Enumeration Commands (important)

### 🔹 `EXPN`

```text
EXPN admin
```

* Expands mailing lists
* Often disabled

---

### 🔹 `RCPT TO` method (most reliable)

```text
MAIL FROM:test@test.com
RCPT TO:admin@example.com
```

Response:

* `250 OK` → user exists
* `550` → user doesn’t exist

📌 Works even when `VRFY` is disabled

---

# 🔔 Blue Team / SOC View

SMTP enumeration triggers:

* Suspicious SMTP commands
* Multiple `VRFY` or `RCPT TO`
* Enumeration patterns

Logged as:

```text
SMTP invalid recipient attempts
```

---

# ⚠️ Legal Warning (VERY IMPORTANT)

> SMTP enumeration without permission is **illegal**.

Only perform on:

* Labs (HTB, TryHackMe)
* Authorized pentests
* Your own mail servers

---

# 🧠 Typical SMTP Enumeration Flow

```text
1. Connect to port 25
2. Read banner
3. Test VRFY
4. Try RCPT TO
5. Collect valid users
```

---

# 🔑 One-Line Takeaway

> **SMTP enumeration exploits mail server responses to discover valid users by actively querying port 25.**