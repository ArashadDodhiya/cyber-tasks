# 🟢 1. Online Password Attacks

These happen **against a live login system** (website, SSH, VPN, etc.).

You send login attempts to the real server.

---

## 🔹 Brute Force Attack

> Try every possible password until one works.

Example:

```
admin : 000001
admin : 000002
admin : 000003
...
```

💡 Works only if:

* No rate limiting
* No account lockout
* Weak password policy

⚠️ Very noisy. Easy to detect.

---

## 🔹 Dictionary Attack

Instead of trying everything,
you try **common passwords** from a wordlist.

Example:

```
password
admin123
welcome
qwerty
letmein
```

Smarter than brute force.

---

## 🔹 Password Spraying (VERY common in real world)

Instead of attacking one user with many passwords…

You try **one common password against many users**.

Example:

```
user1 : Welcome@123
user2 : Welcome@123
user3 : Welcome@123
```

Why this works:

* Avoids account lockout
* Many companies use same default password

🔥 This is very popular in corporate attacks.

---

## 🔹 Credential Stuffing

> Using leaked username/password combos from data breaches.

Example:

* User leaked password from some shopping site
* You try same password on:

  * Gmail
  * LinkedIn
  * Company portal

Works because:
👉 People reuse passwords.

---

# 🔵 2. Offline Password Attacks

This is more dangerous.

You already have:

* A password hash
* A database dump
* An NTLM hash

Now you attack it **offline**, without touching the server.

Server can't detect it.

---

## 🔹 Hash Cracking

If you get something like:

```
5f4dcc3b5aa765d61d8327deb882cf99
```

That’s a hash.

You try to find the original password that produces that hash.

---

## 🔹 Brute Force (Offline)

Try every possible password → hash it → compare.

Fast if:

* Hash is weak (MD5, SHA1)
* No salting

---

## 🔹 Dictionary + Rules Attack

Start with wordlist:

```
password
admin
company
```

Apply rules:

```
password123
Password123!
admin@2025
Company#1
```

Much smarter.

---

## 🔹 Rainbow Table Attack

Precomputed hash tables.

Instead of hashing every time,
you look up the hash in a pre-made table.

⚠️ Doesn’t work well if password is salted.

---

# 🟣 3. Hybrid & Smart Attacks

## 🔹 Mask Attack

If you know password pattern:

```
Company@####
```

You brute force only last 4 digits.

Much faster than full brute force.

---

# 🧠 Social Engineering (Non-Technical)

Sometimes easiest method is:

> Just trick the user.

* Phishing
* Fake login pages
* Calling helpdesk
* Shoulder surfing

Honestly… this works more than technical attacks 😅

---

# 🔐 Why Some Passwords Are Hard to Crack

Good protection includes:

* Strong hashing (bcrypt, Argon2)
* Salt
* Rate limiting
* MFA
* Account lockout
* CAPTCHA

---

# 🧩 Quick Comparison

| Attack Type          | Needs Live Server? | Detectable? |
| -------------------- | ------------------ | ----------- |
| Brute Force (Online) | Yes                | Yes         |
| Password Spraying    | Yes                | Sometimes   |
| Credential Stuffing  | Yes                | Sometimes   |
| Hash Cracking        | No                 | No          |
| Rainbow Table        | No                 | No          |

---

# 🎯 Real-World Pentest Flow

1. Try default credentials
2. Try password spray
3. Look for exposed backups
4. Dump hashes
5. Crack offline
6. Reuse cracked password elsewhere

