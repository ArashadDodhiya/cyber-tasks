# ✅ Lab Prerequisites — Before You Start

> **Run these checks before starting any challenge exercise.**
> If any check FAILS, fix it first using the troubleshooting tips below.

---

## Quick Pre-Flight Checklist

Run these from your **Kali** terminal:

### 1. Kali Internal Network IP
```bash
ip a show eth1
# ✅ EXPECTED: inet 192.168.56.10/24
# ❌ If missing: sudo ip addr add 192.168.56.10/24 dev eth1 && sudo ip link set eth1 up
```

### 2. Metasploitable Is Reachable
```bash
ping -c 2 192.168.56.20
# ✅ EXPECTED: 2 packets transmitted, 2 received, 0% packet loss
# ❌ If fails: Check Metasploitable VM is running & on 'hacklab' internal network
```

### 3. DVWA Is Accessible
```bash
curl -s -o /dev/null -w "%{http_code}" http://192.168.56.20/dvwa/
# ✅ EXPECTED: 200 or 302
# ❌ If fails: On Metasploitable run: sudo service apache2 start
```

Open in Kali Firefox: `http://192.168.56.20/dvwa/` → Login: `admin` / `password`

### 4. DVWA Database Is Set Up
```
1. Go to http://192.168.56.20/dvwa/setup.php
2. Click "Create / Reset Database"
3. Login again: admin / password
```

### 5. DVWA Security Level Is Set to Low (Start Here)
```
1. Go to http://192.168.56.20/dvwa/security.php
2. Set Security Level to "Low"
3. Click Submit
```

### 6. Kali Has Internet (for downloading tools)
```bash
ping -c 2 google.com
# ✅ EXPECTED: replies from google.com
# ❌ If fails: sudo dhclient eth0
```

### 7. Burp Suite Works
```bash
burpsuite &
# ✅ Open Burp → Proxy → Intercept → set browser proxy to 127.0.0.1:8080
# Configure Kali Firefox: Settings → Network → Manual Proxy → 127.0.0.1, port 8080
```

### 8. Essential Tools Available
```bash
# Verify these are installed
which nmap whatweb nikto hashcat john hydra curl nc sqlmap
# ✅ All should return paths
# ❌ If missing: sudo apt install <tool-name> -y
```

---

## Optional: Install OWASP Juice Shop (Needed for Challenges 8 & 9)

```bash
# Install Docker if not already installed
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# Pull and run Juice Shop
sudo docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop

# Verify it's running
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
# ✅ EXPECTED: 200

# Access at: http://localhost:3000
```

---

## Lab Network Summary

```
┌──────────────────────────────────────────────────┐
│            Internal Network: hacklab              │
│                                                   │
│  ┌────────────────┐       ┌───────────────────┐  │
│  │     KALI        │       │  METASPLOITABLE   │  │
│  │ 192.168.56.10   │◄─────►│  192.168.56.20    │  │
│  │                 │       │                    │  │
│  │ Your tools:     │       │ Targets:           │  │
│  │ nmap, burp,     │       │ DVWA (:80/dvwa/)   │  │
│  │ sqlmap, hydra   │       │ FTP (:21)          │  │
│  │                 │       │ SSH (:22)          │  │
│  │ Juice Shop:     │       │ SMTP (:25)         │  │
│  │ localhost:3000   │       │ SMB (:139/445)     │  │
│  └────────────────┘       └───────────────────┘  │
└──────────────────────────────────────────────────┘
```

---

## Ready? Start with [Challenge 1: Encoding/Hashing](./01_lab_encoding_hashing.md) 🚀
