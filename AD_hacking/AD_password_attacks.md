# 🧠 What is a Password Attack in Active Directory?

A **password attack** is when an attacker tries to:

> Guess, steal, or crack user passwords to gain unauthorized access to an AD environment.

In AD environments, passwords are critical because:

* They protect user accounts
* They protect admin accounts
* They protect service accounts
* They protect domain controllers

If passwords fall → the domain can fall.

---

# 🎯 WHY Attackers Target Passwords in AD

Because:

1. AD controls the entire network
2. One compromised admin account = full domain takeover
3. Many organizations still use weak passwords
4. Users reuse passwords
5. Service accounts often have weak or old passwords

Passwords are often the **easiest entry point**.

---

# 📍 WHERE Password Attacks Happen in AD

Password attacks can target:

| Target                  | Example              |
| ----------------------- | -------------------- |
| User accounts           | Regular employee     |
| Privileged accounts     | Domain Admin         |
| Service accounts        | SQL service account  |
| Local admin accounts    | On workstations      |
| Kerberos authentication | Ticket-based attacks |

They can happen:

* Over the network
* On a compromised workstation
* Against a Domain Controller
* Offline (after dumping hashes)

---

# 🔥 Common Types of AD Password Attacks

Let’s go one by one with examples.

---

# 1️⃣ Password Spraying

## 🧠 What It Is

Trying **one common password** against many users.

Instead of:

> Many passwords for one user

They do:

> One password for many users

This avoids account lockout.

---

## 📌 Example

Company policy locks account after 5 failed attempts.

Attacker tries:

Password:

```
Winter2025!
```

Against:

* user1
* user2
* user3
* user4
* user5

If 3 users use that password → attacker gets access.

---

## 🎯 Why It Works

Because:

* Users choose predictable passwords
* Seasonal passwords are common
* Organizations don’t enforce strong complexity

---

# 2️⃣ Brute Force Attack

## 🧠 What It Is

Trying many password combinations for one account.

Example:

```
Admin123
Admin@123
Admin2025
Password1
```

⚠️ Less common in AD because account lockout policies stop this quickly.

---

# 3️⃣ Credential Dumping + Offline Cracking

## 🧠 What It Is

If attacker compromises a machine:

1. Extracts password hashes
2. Takes them offline
3. Attempts cracking

No lockout risk (offline).

---

## 📌 Example Scenario

1. Employee laptop infected.
2. Attacker gets local admin.
3. Extracts NTLM hashes or cached credentials.
4. Uses offline cracking.
5. Password found: `Welcome@123`
6. Logs into domain.

---

# 4️⃣ Pass-the-Hash (PtH)

## 🧠 What It Is

Using NTLM hash directly instead of cracking password.

If attacker gets:

```
NTLM hash of admin
```

They can authenticate without knowing plaintext password.

---

## 📌 Example

1. Attacker dumps memory from compromised machine.
2. Finds Domain Admin NTLM hash.
3. Uses that hash to authenticate to another server.
4. Gains domain control.

---

# 5️⃣ Kerberoasting

## 🧠 What It Is

Targeting service accounts.

Steps (high-level):

1. Request Kerberos service ticket.
2. Ticket contains encrypted data.
3. Extract ticket.
4. Crack offline.

If service account password is weak → attacker recovers it.

---

## 📌 Why Service Accounts Are Dangerous

* Often have:

  * Old passwords
  * Never rotated
  * High privileges
* Sometimes password = company name

---

# 📊 Quick Comparison

| Attack             | Online/Offline | Lockout Risk | Common?     |
| ------------------ | -------------- | ------------ | ----------- |
| Password Spraying  | Online         | Low          | Very common |
| Brute Force        | Online         | High         | Less common |
| Credential Dumping | Offline        | None         | Very common |
| Pass-the-Hash      | Online         | None         | Very common |
| Kerberoasting      | Offline        | None         | Very common |

---

# 🧪 Full Attack Chain Example

Let’s simulate a realistic scenario:

1. Attacker sends phishing email.
2. User machine infected.
3. Attacker performs password spraying.
4. Finds weak password: `Summer2025!`
5. That user has local admin.
6. Dumps hashes.
7. Finds Domain Admin hash.
8. Performs Pass-the-Hash.
9. Domain compromised.

All because of weak passwords.

---

# 🛡️ How to Defend Against AD Password Attacks

## 1️⃣ Strong Password Policy

* 14+ characters
* No seasonal patterns
* Use passphrases

Example:

```
MyDogEatsPizza@Night!
```

---

## 2️⃣ Account Lockout Policy

* Lock after 5 attempts
* Monitor lockouts

---

## 3️⃣ Multi-Factor Authentication (MFA)

Even if password is stolen → attacker blocked.

---

## 4️⃣ Tiered Admin Model

* Domain Admin never logs into workstations
* Separate privileged accounts

---

## 5️⃣ Monitor for:

* Multiple failed logins (Event ID 4625)
* Kerberos ticket abuse
* Unusual authentication patterns
* Login attempts across many users (spraying)

---

# 🎯 Key Takeaway

In Active Directory:

> Password attacks are the most common and effective way attackers gain domain access.

Because:

* Humans choose weak passwords
* Admins reuse passwords
* Service accounts are neglected
* Passwords are everywhere

---

If you'd like, I can next explain:

* 🔍 Real-world AD breach case study
* 📊 How to detect password spraying in logs
* 🧠 How Kerberoasting works technically (simplified)
* 🛡️ Blue team detection lab ideas
* 🧪 Red team lab setup for practicing ethically

Tell me which direction you want to explore.
