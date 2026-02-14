# 📘 PASSWORD CRACKING – COMPLETE DOCUMENTATION

---

# 1️⃣ What is a Password Hash?

A **hash** is a one-way cryptographic function.

```
Password  →  Hash Function  →  Fixed-Length Output
```

Example:

```
password123 → 482c811da5d5b4bc6d497ffa98491e38
```

Important:

* You **cannot reverse** a hash.
* You can only **guess passwords** and compare hashes.

---

# 2️⃣ Common Password Hash Types

---

## 🔹 MD5 (Old & Broken)

Example:

```
5f4dcc3b5aa765d61d8327deb882cf99
```

* 32 hex characters
* Fast
* Easy to crack
* Used in legacy systems

---

## 🔹 SHA1

Example:

```
5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8
```

* 40 hex characters
* Also considered weak

---

## 🔹 bcrypt

Example:

```
$2b$12$KIXQ...
```

* Slow hashing
* Salted
* Much harder to crack

---

## 🔹 NTLM (Windows)

Used in Windows authentication.

Example:

```
8846f7eaee8fb117ad06bdd830b7586c
```

* Unsalted
* Very fast
* Crackable via GPU

---

## 🔹 NetNTLMv2

⚠️ This is **not stored hash** — it's a challenge-response hash captured during authentication (like via Responder).

Used in:

* SMB authentication
* Active Directory environments

Format:

```
username::domain:challenge:response:blob
```

---

# 3️⃣ Cracking NTLM Hashes

Using **Hashcat**

Correct command:

```bash
hashcat -m 1000 -a 0 hash.txt rockyou.txt
```

### 🔎 Breakdown

| Option        | Meaning                      |
| ------------- | ---------------------------- |
| `-m 1000`     | NTLM hash mode               |
| `-a 0`        | Straight attack (dictionary) |
| `hash.txt`    | File containing hash         |
| `rockyou.txt` | Wordlist                     |

---

## 🔥 Hashcat Attack Modes

| Mode | Meaning                |
| ---- | ---------------------- |
| `0`  | Straight (dictionary)  |
| `1`  | Combination            |
| `3`  | Brute force            |
| `6`  | Hybrid wordlist + mask |
| `7`  | Hybrid mask + wordlist |

---

# 4️⃣ Rule-Based Cracking

Rule-based cracking modifies dictionary words automatically.

Example rule:

```
password → Password1
admin → Admin@123
```

Instead of:

* Trying 10 million passwords
* You intelligently modify 10,000 words

---

## Using Rules in Hashcat

```bash
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r rules/best64.rule
```

### What `best64.rule` Does

* Capitalize first letter
* Add numbers
* Add special characters
* Append 1, 123, !

Example transformation:

```
password → Password → Password1 → Password123
```

---

## Why Rule-Based Cracking is Powerful

Because humans:

* Capitalize first letter
* Add birth year
* Add 123
* Add !

Rule engine exploits human psychology.

---

# 5️⃣ Cracking NetNTLMv2

When you capture via:

* Responder
* SMB relay
* LLMNR poisoning

You get something like:

```
john::DOMAIN:112233445566:abcd1234:010100000000000...
```

---

## Hashcat Mode for NetNTLMv2

```bash
hashcat -m 5600 netntlm.txt rockyou.txt
```

| Mode | Meaning   |
| ---- | --------- |
| 5600 | NetNTLMv2 |

---

## Using Rules for NetNTLMv2

```bash
hashcat -m 5600 netntlm.txt rockyou.txt -r best64.rule
```

---

# 6️⃣ Advanced: Mask Attacks

If you know password pattern:

Example:

```
Company uses: Summer2024!
```

You can use:

```bash
hashcat -m 1000 -a 3 hash.txt Summer?d?d?d!
```

| Mask | Meaning      |
| ---- | ------------ |
| ?d   | digit        |
| ?l   | lowercase    |
| ?u   | uppercase    |
| ?s   | special char |

---

# 7️⃣ Using Rules to Conquer NetNTLMv2

Real-world workflow:

1. Capture NetNTLMv2 via Responder
2. Extract hash
3. Run:

```bash
hashcat -m 5600 hash.txt rockyou.txt -r best64.rule
```

4. If fails:

   * Try bigger rule sets
   * Try mask attack
   * Try custom company-based wordlist

---

# 8️⃣ Mimikatz – Credential Dumping

## What is Mimikatz?

Mimikatz is a post-exploitation tool used to extract:

* Plaintext passwords
* NTLM hashes
* Kerberos tickets
* LSA secrets

---

## Common Commands

Start:

```
privilege::debug
```

Dump credentials:

```
sekurlsa::logonpasswords
```

Dump SAM hashes:

```
lsadump::sam
```

Dump NTDS (Domain Controller):

```
lsadump::ntds
```

---

## What Mimikatz Exploits

Windows stores credentials in:

* LSASS process memory
* SAM database
* Active Directory NTDS.dit

If you get SYSTEM/admin access → you can dump hashes.

---

# 9️⃣ Attack Chain Example (Realistic AD Lab)

1. Get low privilege access
2. Run Mimikatz
3. Dump NTLM hash
4. Crack with:

```
hashcat -m 1000
```

5. Reuse password for:

   * RDP
   * SMB
   * Domain admin escalation

---

# 🔥 Comparing NTLM vs NetNTLMv2

| Feature               | NTLM | NetNTLMv2 |
| --------------------- | ---- | --------- |
| Stored in DB          | Yes  | No        |
| Captured over network | No   | Yes       |
| Crackable offline     | Yes  | Yes       |
| Used in AD attacks    | Yes  | Yes       |

---

# 1️⃣0️⃣ Defensive Perspective

Modern protections:

* Password complexity
* Account lockout
* LSASS protection
* Credential Guard
* Multi-factor authentication

---

# 🧠 Key Pentester Insight

Weak passwords fall into patterns:

* CompanyName123
* Welcome@123
* Summer2024!
* Firstname@123