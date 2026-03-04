# Challenge 1: Observation of Encrypted/Hashed/Encoded String

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Identify the type of encoding/hashing and decode or crack the string.

---

## What They Test

You will be given one or more strings and asked to identify whether they are **encoded**, **hashed**, or **encrypted**, and then recover the original value.

---

## Key Concepts

### Encoding vs Hashing vs Encryption

| Type           | Reversible?           | Purpose                | Example           |
| -------------- | --------------------- | ---------------------- | ----------------- |
| **Encoding**   | ✅ Yes (no key needed) | Data representation    | Base64, Hex, URL  |
| **Hashing**    | ❌ No (one-way)        | Integrity verification | MD5, SHA1, SHA256 |
| **Encryption** | ✅ Yes (with key)      | Confidentiality        | AES, RSA, DES     |

---

## How to Identify the Format

### Encoding Formats

| Format             | Characteristics                                                     | Example                                    |
| ------------------ | ------------------------------------------------------------------- | ------------------------------------------ |
| **Base64**         | `A-Za-z0-9+/` chars, ends with `=` or `==`, length is multiple of 4 | `SGVsbG8gV29ybGQ=` → `Hello World`         |
| **Base32**         | `A-Z2-7` chars, ends with `=`, uppercase only                       | `JBSWY3DPEBLW64TMMQ======` → `Hello World` |
| **Hex (Base16)**   | Only `0-9a-fA-F`, always even length                                | `48656c6c6f` → `Hello`                     |
| **URL Encoding**   | `%XX` format for special characters                                 | `%48%65%6c%6c%6f` → `Hello`                |
| **HTML Entities**  | `&#XX;` or `&name;` format                                          | `&#72;&#101;&#108;&#108;&#111;` → `Hello`  |
| **Binary**         | Only `0` and `1`, groups of 8                                       | `01001000 01100101` → `He`                 |
| **Octal**          | Only digits `0-7`                                                   | `110 145 154 154 157` → `Hello`            |
| **ASCII85**        | Printable ASCII chars, often enclosed in `<~ ~>`                    | `<~87cURDZ~>`                              |
| **ROT13**          | Letters shifted 13 positions (a→n, b→o)                             | `Uryyb` → `Hello`                          |
| **ROT47**          | ASCII chars 33-126 shifted 47 positions                             | Covers numbers and symbols too             |
| **Unicode Escape** | `\uXXXX` format                                                     | `\u0048\u0065\u006c\u006c\u006f` → `Hello` |

### Hash Formats

| Hash Type         | Length                                 | Example                                       | Hashcat Mode |
| ----------------- | -------------------------------------- | --------------------------------------------- | ------------ |
| **MD5**           | 32 hex chars                           | `5d41402abc4b2a76b9719d911017c592`            | `-m 0`       |
| **SHA1**          | 40 hex chars                           | `aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d`    | `-m 100`     |
| **SHA256**        | 64 hex chars                           | `2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c...` | `-m 1400`    |
| **SHA512**        | 128 hex chars                          | Very long hex string                          | `-m 1700`    |
| **NTLM**          | 32 hex chars (like MD5)                | `32ed87bdb5fdc5e9cba88547376818d4`            | `-m 1000`    |
| **bcrypt**        | Starts with `$2a$` or `$2b$`, 60 chars | `$2a$10$N9qo8uLOickgx2ZMRZoMye...`            | `-m 3200`    |
| **SHA-512 crypt** | Starts with `$6$`                      | `$6$rounds=5000$salt$hash...`                 | `-m 1800`    |
| **MD5 crypt**     | Starts with `$1$`                      | `$1$salt$hash...`                             | `-m 500`     |
| **MySQL 4.1+**    | Starts with `*`, 40 hex chars          | `*2470C0C06DEE42FD1618BB9900DFG...`           | `-m 300`     |
| **LM Hash**       | 32 hex chars (uppercase letters)       | `AAD3B435B51404EEAAD3B435B51404EE`            | `-m 3000`    |

### Special Formats

| Format                   | Characteristics                                           |
| ------------------------ | --------------------------------------------------------- |
| **JWT (JSON Web Token)** | Three Base64 parts separated by `.` → `eyJhb.eyJzd.SflKx` |
| **SSH Private Key**      | Starts with `-----BEGIN RSA PRIVATE KEY-----`             |
| **PGP**                  | Starts with `-----BEGIN PGP MESSAGE-----`                 |
| **Windows Hash (SAM)**   | Format: `username:RID:LMhash:NTLMhash:::`                 |
| **Linux Shadow**         | Format: `username:$type$salt$hash:...`                    |

---

## Decoding Commands

### Base64
```bash
# Decode
echo "SGVsbG8gV29ybGQ=" | base64 -d

# Encode
echo "Hello World" | base64

# Decode file
base64 -d encoded_file.txt > decoded_file.txt

# Multiple layers of Base64 (sometimes strings are double/triple encoded)
echo "U0dWc2JHOD0=" | base64 -d | base64 -d
```

### Hex
```bash
# Decode hex to text
echo "48656c6c6f" | xxd -r -p

# Decode with Python
python3 -c "print(bytes.fromhex('48656c6c6f').decode())"

# Encode text to hex
echo -n "Hello" | xxd -p
```

### URL Encoding
```bash
# Decode
python3 -c "import urllib.parse; print(urllib.parse.unquote('%48%65%6c%6c%6f'))"

# Decode (alternative)
echo -e '\x48\x65\x6c\x6c\x6f'

# Double URL decode
python3 -c "import urllib.parse; print(urllib.parse.unquote(urllib.parse.unquote('%2548%2565%256c%256c%256f')))"
```

### ROT13
```bash
# Decode/Encode (same operation)
echo "Uryyb" | tr 'A-Za-z' 'N-ZA-Mn-za-m'

# Python
python3 -c "import codecs; print(codecs.decode('Uryyb', 'rot_13'))"
```

### Binary / Octal
```bash
# Binary to text
python3 -c "print(''.join(chr(int(b, 2)) for b in '01001000 01100101 01101100 01101100 01101111'.split()))"

# Octal to text
python3 -c "print(''.join(chr(int(o, 8)) for o in '110 145 154 154 157'.split()))"
```

### JWT
```bash
# Decode JWT header (first part before .)
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" | base64 -d 2>/dev/null

# Decode JWT payload (second part)
echo "eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ" | base64 -d 2>/dev/null

# Online: https://jwt.io
```

---

## Hash Identification

### Using Tools
```bash
# hash-identifier
hash-identifier
# Paste the hash when prompted

# hashid (more accurate)
hashid '5d41402abc4b2a76b9719d911017c592'
hashid -m '5d41402abc4b2a76b9719d911017c592'  # Shows hashcat mode

# haiti
haiti '5d41402abc4b2a76b9719d911017c592'

# name-that-hash (most modern)
nth -t '5d41402abc4b2a76b9719d911017c592'
```

### Quick Manual Identification
```
32 chars hex  → MD5 or NTLM
40 chars hex  → SHA1 or MySQL
64 chars hex  → SHA256
128 chars hex → SHA512
$1$           → MD5 crypt
$2a$ or $2b$  → bcrypt
$5$           → SHA-256 crypt
$6$           → SHA-512 crypt
$y$           → yescrypt (modern Linux)
```

---

## Hash Cracking

### Hashcat
```bash
# MD5
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# SHA1
hashcat -m 100 hash.txt /usr/share/wordlists/rockyou.txt

# SHA256
hashcat -m 1400 hash.txt /usr/share/wordlists/rockyou.txt

# NTLM
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt

# bcrypt
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt

# With rules (more combinations)
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Show cracked results
hashcat -m 0 hash.txt --show
```

### John the Ripper
```bash
# Auto-detect hash type
john hash.txt

# With wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Show cracked passwords
john --show hash.txt

# Specific format
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### Online Tools
```
https://crackstation.net         - Free hash cracker (large database)
https://hashes.com               - Hash lookup
https://www.md5online.org        - MD5 specific
https://hashtoolkit.com          - Multiple hash types
```

---

## Multi-Layer Encoding (Common in CTFs)

Sometimes strings are encoded multiple times. Follow this approach:

```
1. Identify outermost encoding
2. Decode one layer
3. Repeat until plaintext appears

Example: A string encoded as Base64 → Hex → ROT13
Step 1: Decode Base64
Step 2: Decode Hex  
Step 3: Decode ROT13
Result: Plaintext!
```

### CyberChef (Best Tool for Multi-Layer)
```
URL: https://gchq.github.io/CyberChef/
- Drag operations to the "Recipe" area
- Use "Magic" operation for auto-detection
- Chain multiple operations together
```

---

## Practice Scenarios

| Scenario                    | What You Receive                   | Expected Action                     |
| --------------------------- | ---------------------------------- | ----------------------------------- |
| Base64 string in cookie     | `YWRtaW46cGFzc3dvcmQ=`             | Decode → `admin:password`           |
| Hash in database dump       | `5f4dcc3b5aa765d61d8327deb882cf99` | Identify as MD5, crack → `password` |
| JWT in Authorization header | `eyJhbGci...`                      | Decode header + payload             |
| Hex in URL parameter        | `2f6574632f706173737764`           | Decode → `/etc/passwd`              |
| Double-encoded string       | `JTI1NDglMjU2NSUyNTZj`             | URL decode twice                    |

---

## Tips for the Interview

1. **Always try CyberChef's "Magic" operation first** — it auto-detects encoding
2. **Check hash length immediately** — 32 chars = MD5/NTLM, 40 = SHA1, 64 = SHA256
3. **Look for patterns**: `$2a$` = bcrypt, `eyJ` = JWT, `==` ending = Base64
4. **Try crackstation.net first** for hash cracking — it's instant for common passwords
5. **Don't forget double/triple encoding** — decode iteratively
