# 🔬 Lab Exercise 1: Observation of Encrypted/Hashed/Encoded String

> **Challenge:** Identify encoding/hash types and decode or crack them
> **Where:** Kali Linux terminal (no target VM needed)
> **Time:** ~30-45 minutes

---

## Exercise 1A: Identify the Encoding Type

**Task:** For each string below, identify the encoding type and decode it on your Kali terminal.

### String 1 — Base64
```bash
# This is your sample string:
echo "Y3liZXJfc2VjdXJpdHlfcm9ja3M=" | base64 -d
# ✅ EXPECTED OUTPUT: cyber_security_rocks
```

### String 2 — Hex
```bash
# Decode this hex string:
echo "70617373776f72643132333435" | xxd -r -p
# ✅ EXPECTED OUTPUT: password12345
```

### String 3 — URL Encoding
```bash
# Decode this URL-encoded string:
python3 -c "import urllib.parse; print(urllib.parse.unquote('%2Fetc%2Fpasswd'))"
# ✅ EXPECTED OUTPUT: /etc/passwd
```

### String 4 — ROT13
```bash
# Decode this ROT13 string:
echo "frperg_cnffjbeq" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
# ✅ EXPECTED OUTPUT: secret_password
```

### String 5 — Double Encoded (Base64 inside Base64)
```bash
# This is double-encoded Base64:
echo "WVdSdGFXNDZjR0Z6YzNkdmNtUT0=" | base64 -d | base64 -d
# ✅ EXPECTED OUTPUT: admin:password
```

### String 6 — Binary
```bash
# Decode this binary string:
python3 -c "print(''.join(chr(int(b, 2)) for b in '01110010 01101111 01101111 01110100'.split()))"
# ✅ EXPECTED OUTPUT: root
```

---

## Exercise 1B: Identify Hash Types

**Task:** Use `hashid` or `hash-identifier` to identify each hash, then crack it.

### Hash 1 — MD5
```bash
# Step 1: Identify the hash type
hashid '5f4dcc3b5aa765d61d8327deb882cf99'
# ✅ EXPECTED: [+] MD5

# Step 2: Crack it with hashcat
echo "5f4dcc3b5aa765d61d8327deb882cf99" > /tmp/hash1.txt
hashcat -m 0 /tmp/hash1.txt /usr/share/wordlists/rockyou.txt --force
# ✅ EXPECTED: 5f4dcc3b5aa765d61d8327deb882cf99:password

# Alternative: Crack with John
echo "5f4dcc3b5aa765d61d8327deb882cf99" > /tmp/hash1_john.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt /tmp/hash1_john.txt
john --format=raw-md5 --show /tmp/hash1_john.txt
# ✅ EXPECTED: ?:password
```

> **Note:** First time using rockyou.txt? Decompress it:
> `sudo gunzip /usr/share/wordlists/rockyou.txt.gz`

### Hash 2 — SHA1
```bash
# Step 1: Identify
hashid 'aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d'
# ✅ EXPECTED: [+] SHA-1

# Step 2: Crack
echo "aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d" > /tmp/hash2.txt
hashcat -m 100 /tmp/hash2.txt /usr/share/wordlists/rockyou.txt --force
# ✅ EXPECTED: aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d:hello
```

### Hash 3 — SHA256
```bash
# Step 1: Identify
hashid '8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918'
# ✅ EXPECTED: [+] SHA-256

# Step 2: Crack
echo "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918" > /tmp/hash3.txt
hashcat -m 1400 /tmp/hash3.txt /usr/share/wordlists/rockyou.txt --force
# ✅ EXPECTED: ....:admin
```

### Hash 4 — NTLM
```bash
# Step 1: Identify (looks like MD5 but it's NTLM!)
hashid '32ed87bdb5fdc5e9cba88547376818d4'
# EXPECTED: Both MD5 and NTLM will show — in real scenarios, context helps

# Step 2: Crack as NTLM
echo "32ed87bdb5fdc5e9cba88547376818d4" > /tmp/hash4.txt
hashcat -m 1000 /tmp/hash4.txt /usr/share/wordlists/rockyou.txt --force
```

---

## Exercise 1C: Create Your Own Hashes (Verify You Understand)

```bash
# Create an MD5 hash of "cybersecurity"
echo -n "cybersecurity" | md5sum
# 📝 Write down the hash: ___________________________

# Create a SHA256 hash of "cybersecurity"
echo -n "cybersecurity" | sha256sum
# 📝 Write down the hash: ___________________________

# Create a SHA1 hash of "interview2024"
echo -n "interview2024" | sha1sum
# 📝 Write down the hash: ___________________________

# Create a Linux password hash
openssl passwd -1 -salt testsalt mypassword123
# 📝 Write down the hash: ___________________________
# 📝 What does the $1$ at the beginning tell you? ___________
```

---

## Exercise 1D: JWT Decoding

```bash
# Here's a sample JWT token:
JWT="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"

# Decode the header (part 1)
echo "$JWT" | cut -d '.' -f 1 | base64 -d 2>/dev/null; echo
# ✅ EXPECTED: {"alg":"HS256","typ":"JWT"}

# Decode the payload (part 2)
echo "$JWT" | cut -d '.' -f 2 | base64 -d 2>/dev/null; echo
# ✅ EXPECTED: {"sub":"1234567890","name":"John Doe","iat":1516239022}

# Part 3 is the signature — it can't be decoded without the secret key
```

---

## Exercise 1E: Multi-Layer Decoding with CyberChef

**Task:** Open CyberChef in Kali Firefox and practice:

```
1. Go to: https://gchq.github.io/CyberChef/
2. Paste this string in the Input box:
   NTI1NTY4NjU3MjczNjU2Mzc1NzI2OTc0Nzk=
3. Add "From Base64" operation → drag it to Recipe
4. See output: 5255686572736563757269747
   → This looks like Hex!
5. Add "From Hex" operation → drag to Recipe
6. See final output: Rusecurity
```

Also try CyberChef's **Magic** button — it auto-detects encoding!

---

## Exercise 1F: Real-World Scenario — Crack Metasploitable Hashes

**Task:** Get real password hashes from Metasploitable and crack them.

```bash
# Step 1: SSH into Metasploitable
ssh msfadmin@192.168.56.20
# Password: msfadmin

# Step 2: View the shadow file (Metasploitable has weak permissions!)
cat /etc/shadow
# 📝 Copy the hashes you see (lines starting with a username)

# Step 3: Exit back to Kali
exit

# Step 4: On Kali, save a hash to crack (e.g., msfadmin's hash)
# Copy the full line from /etc/shadow for msfadmin
# It will look like: msfadmin:$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/:14684:0:99999:7:::

# Step 5: Create unshadow file
scp msfadmin@192.168.56.20:/etc/passwd /tmp/ms_passwd
scp msfadmin@192.168.56.20:/etc/shadow /tmp/ms_shadow
unshadow /tmp/ms_passwd /tmp/ms_shadow > /tmp/combined.txt

# Step 6: Crack with John
john /tmp/combined.txt --wordlist=/usr/share/wordlists/rockyou.txt
john --show /tmp/combined.txt
# ✅ EXPECTED: You should crack msfadmin's password and possibly others!
```

---

## ✅ Completion Checklist

- [ ] Decoded all 6 encoded strings (1A)
- [ ] Identified and cracked 4 hash types (1B)
- [ ] Created your own hashes and understood format (1C)
- [ ] Decoded a JWT token (1D)
- [ ] Used CyberChef for multi-layer decoding (1E)
- [ ] Cracked real hashes from Metasploitable (1F)

---

**Next:** [Challenge 2: Web Time Traveller →](./02_lab_web_time_traveller.md)
