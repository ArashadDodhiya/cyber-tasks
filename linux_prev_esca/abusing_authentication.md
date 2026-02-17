# 🧠 1️⃣ How Linux Password Authentication Works (Simple)

When you log in:

```
login → system checks username → verifies password hash → grants access
```

The important files involved:

```
/etc/passwd
/etc/shadow
```

---

# 📂 2️⃣ /etc/passwd Explained

View it:

```bash
cat /etc/passwd
```

Example line:

```
john:x:1001:1001:John Doe:/home/john:/bin/bash
```

Format:

```
username : password_placeholder : UID : GID : comment : home_dir : shell
```

Important parts:

| Field    | Meaning                        |
| -------- | ------------------------------ |
| username | Login name                     |
| x        | Password stored in /etc/shadow |
| UID      | User ID                        |
| GID      | Group ID                       |
| home_dir | User home                      |
| shell    | Login shell                    |

---

### 🔑 Why “x”?

Originally, password hashes were stored in `/etc/passwd`.

But since `/etc/passwd` is world-readable, that was insecure.

So Linux moved hashes to:

```
/etc/shadow
```

---

# 🔒 3️⃣ /etc/shadow Explained

View (only root can):

```bash
cat /etc/shadow
```

Example:

```
john:$6$kjsdfhksdjf$Aksjdhfksjdhf...:19000:0:99999:7:::
```

Format:

```
username : password_hash : last_change : min : max : warn : inactive : expire
```

Important part:

```
$6$ = SHA-512 hash
```

This is the encrypted password.

---

# 🚨 How These Files Are Abused (In Labs / CTFs)

Now let’s talk about common misconfigurations.

---

# 🔥 1️⃣ Writable /etc/passwd (Very Serious)

If:

```
/etc/passwd
```

Is writable by normal user:

```
-rw-rw-rw-
```

That’s critical.

---

### Why is this dangerous?

You can:

* Change your UID to 0 (root)
* Or add a new root user

Example lab abuse:

Change:

```
john:x:1001:1001:...
```

To:

```
john:x:0:0:...
```

Now john becomes root.

Or add:

```
hacker:x:0:0:root:/root:/bin/bash
```

Then login as hacker.

⚠️ This only works if the file is writable.

---

# 🔥 2️⃣ Writable /etc/shadow

Even worse.

If writable, attacker can:

* Replace root password hash
* Set empty password
* Insert known hash

That gives full root login.

This is extremely rare in real systems.

---

# 🔥 3️⃣ Readable /etc/shadow

If normal user can read:

```
cat /etc/shadow
```

Attacker can:

1. Copy password hashes
2. Crack them offline using tools (in lab environment)
3. Recover weak passwords

This is called:

> Password hash dumping + offline cracking

In real-world security assessments, this demonstrates weak password policies.

---

# 🔥 4️⃣ Empty Passwords

In `/etc/shadow`:

If you see:

```
john::19000:0:99999:7:::
```

No hash = empty password.

That account might allow login without password.

---

# 🔥 5️⃣ Weak Hash Algorithm

Old systems may use:

```
$1$  → MD5
```

Instead of:

```
$6$ → SHA-512
```

MD5 hashes are much easier to crack.

---

# 🛡️ Defensive Perspective (Very Important)

To secure these files:

### Correct permissions:

```
/etc/passwd → 644
/etc/shadow → 640 or 600
```

Check with:

```bash
ls -l /etc/passwd
ls -l /etc/shadow
```

Shadow should only be readable by root.

---

# 🧠 Common PrivEsc Exam Scenario

In CTFs, you may find:

* Writable `/etc/passwd`
* Backup file like `/etc/passwd.bak`
* Credentials inside scripts
* Misconfigured NFS exposing shadow

These are intentional vulnerabilities for learning.

---

# 🏆 Real-World Reality

In modern enterprise systems:

* Direct `/etc/passwd` abuse is rare
* Systems use PAM, LDAP, Active Directory
* File permissions are hardened

More common real-world issues:

* Weak passwords
* Reused credentials
* Credential leaks in config files
* Poor sudo configuration

---

# 🎯 Key Takeaway

Abuse of `/etc/passwd` or `/etc/shadow` happens when:

1. File permissions are misconfigured
2. Files are readable or writable
3. Weak hashes allow cracking
4. Empty passwords exist

It’s not about “hacking the file.”
It’s about exploiting poor system configuration.

---

# ⚠️ Ethical Reminder

Only test these things:

* On your own lab
* In CTF platforms
* During authorized penetration tests

Never modify authentication files on production systems.

---

If you want, next I can explain:

* How to detect weak password policies
* How PAM works internally
* How modern Linux authentication differs from Windows
* A step-by-step CTF example involving passwd abuse

Tell me your learning goal 🔐
