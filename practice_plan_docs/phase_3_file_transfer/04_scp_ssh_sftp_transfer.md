# 🔐 SCP / SSH / SFTP Transfer — Step-by-Step Guide

> **What is it?** These methods use SSH (the encrypted remote login protocol) to transfer files. Everything is encrypted, so it's the **most secure** file transfer method.

> **When to use?** When you have SSH credentials (username + password) for the target machine. This is the **safest** method — all data is encrypted.

---

## 📋 What You Need

| Item                  | Details                                                   |
| --------------------- | --------------------------------------------------------- |
| **Kali IP**           | `192.168.56.10`                                           |
| **Metasploitable IP** | `192.168.56.20`                                           |
| **SSH credentials**   | `msfadmin` / `msfadmin` (Metasploitable default)          |
| **Tools needed**      | `scp`, `sftp`, `ssh` — all pre-installed on both machines |

---

## 🧠 SCP vs SFTP vs SSH Piping — When to Use What?

| Method         | Best For                           | How It Works                   |
| -------------- | ---------------------------------- | ------------------------------ |
| **SCP**        | Quick single file or folder copy   | One command, done              |
| **SFTP**       | Browsing + multiple file transfers | Interactive session (like FTP) |
| **SSH Piping** | Piping data without saving to disk | Send data through SSH tunnel   |

---

## 📤 Method 1: SCP — Copy TO Target

> **SCP** = Secure Copy. One command to copy a file.

### Send a file from Kali to Metasploitable

```bash
scp test_file.txt msfadmin@192.168.56.20:/tmp/
```

> **Breaking it down:**
> - `test_file.txt` = the file you're sending
> - `msfadmin@192.168.56.20` = username@target_IP
> - `:/tmp/` = where to put it on the target
> - You'll be prompted for the password: `msfadmin`

### Verify on Metasploitable

```bash
ssh msfadmin@192.168.56.20 "cat /tmp/test_file.txt"
# Should show: "This is a test file from Kali"
```

---

## 📥 Method 2: SCP — Copy FROM Target

> Grab a file from the target back to Kali.

### Download a file from Metasploitable to Kali

```bash
scp msfadmin@192.168.56.20:/etc/passwd ./grabbed_passwd.txt
```

> **Breaking it down:**
> - `msfadmin@192.168.56.20:/etc/passwd` = file on the target
> - `./grabbed_passwd.txt` = save it here on Kali
> - Password: `msfadmin`

### Verify on Kali

```bash
cat grabbed_passwd.txt
# Shows Metasploitable's /etc/passwd file
```

---

## 📁 Method 3: SCP — Copy Entire Directory

```bash
# Create a tools folder with multiple files for testing
mkdir -p ~/tools
echo "tool1 content" > ~/tools/tool1.sh
echo "tool2 content" > ~/tools/tool2.sh

# Copy entire folder to target
scp -r ~/tools/ msfadmin@192.168.56.20:/tmp/
```

> **`-r` flag** = recursive (copy the folder and everything inside).

### Verify

```bash
ssh msfadmin@192.168.56.20 "ls -la /tmp/tools/"
# Should show tool1.sh and tool2.sh
```

---

## 📤 Method 4: SCP with Non-Standard SSH Port

> Some machines run SSH on a different port (not 22).

```bash
# If SSH runs on port 2222 instead of 22
scp -P 2222 test_file.txt msfadmin@192.168.56.20:/tmp/
```

> ⚠️ Note: SCP uses **uppercase** `-P` for port (SSH uses lowercase `-p`).

---

## 📂 Method 5: SFTP — Interactive File Transfer

> **SFTP** = SSH File Transfer Protocol. It gives you an interactive session where you can browse directories, download, and upload — like FTP but encrypted.

### Step 1: Connect

```bash
sftp msfadmin@192.168.56.20
# Password: msfadmin
```

### Step 2: Use SFTP commands

```bash
# You're now in the SFTP shell
sftp> pwd                    # Show current remote directory
sftp> lpwd                   # Show current LOCAL directory
sftp> ls                     # List remote files
sftp> lls                    # List LOCAL files
sftp> cd /tmp                # Change remote directory
sftp> lcd ~/Documents        # Change LOCAL directory

# Download a file FROM target
sftp> get /etc/passwd

# Upload a file TO target
sftp> put test_file.txt /tmp/test_file.txt

# Download multiple files
sftp> mget *.txt

# Upload multiple files
sftp> mput *.sh

# Exit
sftp> quit
```

### SFTP Command Summary

| Command        | What It Does                    |
| -------------- | ------------------------------- |
| `pwd` / `lpwd` | Print remote / local directory  |
| `ls` / `lls`   | List remote / local files       |
| `cd` / `lcd`   | Change remote / local directory |
| `get file`     | Download file from target       |
| `put file`     | Upload file to target           |
| `mget *.txt`   | Download multiple files         |
| `mput *.sh`    | Upload multiple files           |
| `quit`         | Exit SFTP                       |

> **Tip**: Commands starting with `l` (like `lls`, `lcd`, `lpwd`) work on your **local** machine.

---

## 🔗 Method 6: SSH Piping (Advanced)

> Send file contents through SSH without using SCP or SFTP. Useful when you want to pipe data.

### Pipe a file TO the target

```bash
cat test_file.txt | ssh msfadmin@192.168.56.20 "cat > /tmp/test_file.txt"
```

> **What happens:**
> - `cat test_file.txt` reads the file on Kali
> - `|` pipes the content through SSH
> - `cat > /tmp/test_file.txt` saves it on the target

### Pipe a file FROM the target

```bash
ssh msfadmin@192.168.56.20 "cat /etc/shadow" > shadow.txt
```

> This grabs `/etc/shadow` from Metasploitable and saves it as `shadow.txt` on Kali.

### Pipe and compress at the same time

```bash
# Compress and transfer a directory
ssh msfadmin@192.168.56.20 "tar czf - /etc/" > etc_backup.tar.gz
```

> This creates a compressed archive of `/etc/` on the fly and saves it on Kali.

---

## 🧠 Quick Reference Cheatsheet

| What You Want             | Command                                          |
| ------------------------- | ------------------------------------------------ |
| **Copy file TO target**   | `scp file.txt user@TARGET:/path/`                |
| **Copy file FROM target** | `scp user@TARGET:/path/file.txt ./`              |
| **Copy folder TO target** | `scp -r folder/ user@TARGET:/path/`              |
| **SCP with custom port**  | `scp -P PORT file.txt user@TARGET:/path/`        |
| **SFTP session**          | `sftp user@TARGET`                               |
| **Pipe file TO target**   | `cat file \| ssh user@TARGET "cat > /path/file"` |
| **Pipe file FROM target** | `ssh user@TARGET "cat /path/file" > local_file`  |

---

## ❗ Common Mistakes & Fixes

| Problem                        | Fix                                                     |
| ------------------------------ | ------------------------------------------------------- |
| `Connection refused`           | SSH not running on target, or wrong port                |
| `Permission denied`            | Wrong username/password, or file permissions            |
| `No such file or directory`    | Check the remote path exists                            |
| `Host key verification failed` | Run `ssh-keygen -R TARGET_IP` to reset                  |
| SCP is very slow               | Could be DNS lookup — add `-o StrictHostKeyChecking=no` |

---

## ⚖️ SCP/SSH vs Other Methods

| Feature           | SCP/SSH/SFTP     | HTTP            | Netcat   |
| ----------------- | ---------------- | --------------- | -------- |
| Encryption        | ⭐⭐⭐ Yes (SSH)    | ❌ No            | ❌ No     |
| Needs credentials | Yes (SSH login)  | No              | No       |
| Browse files      | SFTP: Yes        | HTTP: Yes       | No       |
| Stealth           | High (encrypted) | Low (logs)      | Medium   |
| Best for          | Secure transfers | Quick downloads | Raw data |

---

## ✅ Practice Checklist

- [ ] SCP a file from Kali to Metasploitable `/tmp/`
- [ ] SCP a file from Metasploitable back to Kali
- [ ] SCP an entire directory with `-r`
- [ ] Open an SFTP session and browse around
- [ ] Download a file using SFTP `get`
- [ ] Upload a file using SFTP `put`
- [ ] Try SSH piping to grab `/etc/shadow`

---

> 🔙 **Previous**: [SMB Transfer](./03_smb_transfer.md)
> 🔙 **Back to**: [File Transfer Main Guide](./file_transfer.md)
> ➡️ **Next**: [FTP / TFTP Transfer](./05_ftp_tftp_transfer.md)
