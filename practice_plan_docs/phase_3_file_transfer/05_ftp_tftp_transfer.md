# 📡 FTP / TFTP Transfer — Step-by-Step Guide

> **What is it?** FTP (File Transfer Protocol) and TFTP (Trivial FTP) are old-school file transfer protocols. FTP gives you an interactive session with login; TFTP is simpler with no authentication.

> **When to use?** When the target has an FTP server running (Metasploitable has one!) or when you need a simple file server without SSH.

---

## 📋 What You Need

| Item                  | Details                                                       |
| --------------------- | ------------------------------------------------------------- |
| **Kali IP**           | `192.168.56.10`                                               |
| **Metasploitable IP** | `192.168.56.20`                                               |
| **FTP credentials**   | `msfadmin` / `msfadmin` (or `anonymous` if allowed)           |
| **Tools**             | `ftp` client (pre-installed), `pyftpdlib` (Python FTP server) |

---

## 🧠 FTP vs TFTP — What's the Difference?

| Feature            | FTP                     | TFTP                    |
| ------------------ | ----------------------- | ----------------------- |
| Authentication     | Yes (username/password) | No (anonymous)          |
| Browse directories | Yes                     | No                      |
| Port               | 21                      | 69                      |
| Encryption         | ❌ None                  | ❌ None                  |
| Best for           | Full file management    | Quick, simple transfers |

---

## 📥 Method 1: Connect to Metasploitable's FTP Server

> Metasploitable has an FTP server pre-installed. Let's connect to it.

### Step 1: Connect from Kali

```bash
ftp 192.168.56.20
```

### Step 2: Login

```
Name: msfadmin
Password: msfadmin
```

> You should see: `230 Login successful.`

### Step 3: Browse and download files

```bash
ftp> pwd                    # Show current directory
ftp> ls                     # List files
ftp> cd /tmp                # Change to /tmp
ftp> get /etc/passwd        # Download a file
ftp> quit                   # Exit
```

### Step 4: Verify

```bash
cat passwd
# Shows Metasploitable's /etc/passwd
```

---

## 📤 Method 2: Upload Files via FTP

### While connected to FTP

```bash
ftp> put test_file.txt      # Upload file to target
ftp> ls                     # Verify it's there
```

> ⚠️ Upload may fail if the FTP user doesn't have write permissions in the current directory. Try `cd /tmp` first.

---

## 📤 Method 3: Binary Mode (For Executables/Images)

> **Important!** FTP defaults to ASCII mode, which can **corrupt binary files** (executables, images, archives). Always switch to binary mode first.

```bash
ftp> binary                 # Switch to binary transfer mode
ftp> get some_executable    # Now it transfers correctly
ftp> ascii                  # Switch back to ASCII (for text files)
```

> **Rule of thumb:**
> - Text files (`.txt`, `.sh`, `.py`) → ASCII mode is fine
> - Everything else (`.exe`, `.zip`, `.tar.gz`, images) → Use **binary** mode

---

## 📥 Method 4: Start Your Own FTP Server on Kali

> Instead of connecting to the target's FTP server, run your own on Kali. The target then connects to YOU.

### Step 1: Install pyftpdlib (if not installed)

```bash
pip3 install pyftpdlib
```

### Step 2: Start FTP server on Kali

```bash
cd ~/Documents/file_transfer_practice

# Start writable FTP server
sudo python3 -m pyftpdlib -p 21 -w
```

> **Flags:**
> - `-p 21` = run on port 21 (standard FTP port)
> - `-w` = allow write access (so target can upload files to you)

You'll see:

```
[I] Starting FTP server on 0.0.0.0:21
[I] Concurrency model: async
```

### Step 3: On Metasploitable — connect to Kali's FTP server

```bash
ftp 192.168.56.10
# Login: anonymous (default for pyftpdlib)
# Password: (just press Enter)
```

### Step 4: Download files from Kali

```bash
ftp> get test_file.txt      # Download file from Kali
ftp> quit
```

### Step 5: Verify

```bash
cat test_file.txt
# Shows: "This is a test file from Kali"
```

---

## 📥 Method 5: Anonymous FTP Login

> Some FTP servers allow **anonymous** login (no real password needed).

```bash
ftp 192.168.56.20
# Name: anonymous
# Password: (anything or just press Enter)
```

> Metasploitable's vsftpd may or may not allow this. Try it!

---

## 📡 Method 6: TFTP Transfer

> TFTP is even simpler than FTP — no login, no directory browsing. Just get/put files.

### Step 1: Install and start TFTP server on Kali

```bash
# Install TFTP server
sudo apt install atftpd -y

# Create directory for TFTP files
sudo mkdir -p /tmp/tftp
sudo chmod 777 /tmp/tftp

# Copy your test file there
cp ~/Documents/file_transfer_practice/test_file.txt /tmp/tftp/

# Start TFTP server
sudo atftpd --daemon --port 69 /tmp/tftp
```

### Step 2: On Metasploitable — download via TFTP

```bash
# Interactive mode
tftp 192.168.56.10
tftp> get test_file.txt
tftp> quit

# OR one-liner
tftp 192.168.56.10 -c get test_file.txt
```

### Step 3: Upload via TFTP

```bash
tftp 192.168.56.10
tftp> put /etc/hostname
tftp> quit
```

---

## 🧠 Quick Reference Cheatsheet

### FTP Commands

| What You Want             | Command                   |
| ------------------------- | ------------------------- |
| **Connect to FTP server** | `ftp TARGET_IP`           |
| **Login**                 | Enter username + password |
| **List files**            | `ls`                      |
| **Download file**         | `get filename`            |
| **Upload file**           | `put filename`            |
| **Binary mode**           | `binary`                  |
| **Change directory**      | `cd /path`                |
| **Exit**                  | `quit` or `bye`           |

### FTP Server Commands (on Kali)

| What You Want         | Command                                    |
| --------------------- | ------------------------------------------ |
| **Start FTP server**  | `sudo python3 -m pyftpdlib -p 21 -w`       |
| **Start TFTP server** | `sudo atftpd --daemon --port 69 /tmp/tftp` |

---

## ❗ Common Mistakes & Fixes

| Problem                     | Fix                                                  |
| --------------------------- | ---------------------------------------------------- |
| `Connection refused`        | FTP server not running on target                     |
| `530 Login incorrect`       | Wrong username/password                              |
| `553 Could not create file` | No write permission — try `cd /tmp`                  |
| Binary file is corrupted    | Forgot to type `binary` before transfer              |
| `ftp: command not found`    | Install: `sudo apt install ftp`                      |
| TFTP `timeout`              | TFTP server not running or firewall blocking port 69 |

---

## ⚖️ FTP vs Other Methods

| Feature        | FTP             | TFTP              | SCP             | HTTP          |
| -------------- | --------------- | ----------------- | --------------- | ------------- |
| Authentication | Yes             | No                | Yes (SSH)       | No            |
| Encryption     | ❌ No            | ❌ No              | ✅ Yes           | ❌ No          |
| Browse files   | Yes             | No                | No (SFTP:Yes)   | Yes           |
| Ease of setup  | Medium          | Easy              | Easy            | Easy          |
| Best for       | File management | Quick single file | Secure transfer | Serving files |

---

## ✅ Practice Checklist

- [ ] Connect to Metasploitable's FTP server with `msfadmin` credentials
- [ ] Download a file using `get`
- [ ] Upload a file using `put`
- [ ] Switch to binary mode and transfer a non-text file
- [ ] Start your own FTP server on Kali using `pyftpdlib`
- [ ] Connect from Metasploitable to your Kali FTP server
- [ ] Try TFTP transfer (if TFTP is available)

---

> 🔙 **Previous**: [SCP / SSH / SFTP Transfer](./04_scp_ssh_sftp_transfer.md)
> 🔙 **Back to**: [File Transfer Main Guide](./file_transfer.md)
> ➡️ **Next**: [Encoding & Stealth Transfer](./06_encoding_stealth_transfer.md)
