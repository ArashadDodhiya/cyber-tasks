# ✅ Lab Prerequisites — Moderate Challenges

> **Run these checks before starting any moderate challenge exercise.**
> These build on the easy lab setup — your Kali + Metasploitable homelab must already work.

---

## Quick Pre-Flight Checklist

### 1. Kali ↔ Metasploitable Network
```bash
ping -c 2 192.168.56.20
# ✅ EXPECTED: 2 packets transmitted, 2 received, 0% packet loss
```

### 2. DVWA Is Accessible
```bash
curl -s -o /dev/null -w "%{http_code}" http://192.168.56.20/dvwa/
# ✅ EXPECTED: 200 or 302
# Login: admin / password
```

### 3. DVWA Security Level — Start at Low, Then Increase
```
1. Go to http://192.168.56.20/dvwa/security.php
2. Set Security Level to "Low" (start here for each exercise)
3. Labs will instruct you to increase to Medium/High as you progress
```

### 4. OWASP Juice Shop Running
```bash
# Check if Juice Shop container is running
sudo docker ps | grep juice-shop
# ✅ EXPECTED: bkimminich/juice-shop container listed

# If not running:
sudo docker start juice-shop
# Or fresh install:
sudo docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop

# Verify
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
# ✅ EXPECTED: 200
```

### 5. Burp Suite Ready
```bash
burpsuite &
# Configure Kali Firefox proxy: 127.0.0.1:8080
# Proxy → Intercept → OFF (turn ON when exercises say so)
```

### 6. Essential Tools Installed
```bash
# Verify all tools for moderate challenges
which nmap hydra ffuf hashcat john sqlmap curl nc whatweb nikto
# ✅ All should return paths

# Install any missing tools
sudo apt install hydra ffuf -y
```

### 7. Kali Has Internet
```bash
ping -c 2 google.com
# ✅ EXPECTED: replies from google.com
```

---

## Additional Setup for Specific Labs

### For XXE Lab (Challenge 25) — Python Flask Vulnerable Server
```bash
# Install Flask
pip3 install flask lxml

# You'll create a vulnerable XML server during the lab
# No pre-setup needed — the lab provides the script
```

### For Mobile Android Lab (Challenge 17) — Optional
```bash
# Option 1: Android Studio + Emulator (recommended)
# Download from: https://developer.android.com/studio
# Install and create an AVD (Android Virtual Device)

# Option 2: Use apktool/jadx for static analysis only (no emulator needed)
sudo apt install apktool -y
# Download jadx: https://github.com/skylot/jadx/releases

# Download DIVA (Damn Insecure and Vulnerable App):
# https://github.com/payatu/diva-android/releases
# Save APK to ~/Downloads/diva-beta.apk
```

### For Mobile iOS Lab (Challenge 18) — Optional
```bash
# Install MobSF (Mobile Security Framework) via Docker
sudo docker run -d -p 8000:8000 --name mobsf opensecurity/mobile-security-framework-mobsf
# Access at: http://localhost:8000

# Download DVIA-v2 IPA for analysis:
# https://github.com/prateek147/DVIA-v2/releases
```

---

## Lab Network Summary

```
┌──────────────────────────────────────────────────────┐
│              Internal Network: hacklab                │
│                                                       │
│  ┌────────────────────┐     ┌───────────────────┐    │
│  │       KALI          │     │  METASPLOITABLE   │    │
│  │  192.168.56.10      │◄───►│  192.168.56.20    │    │
│  │                     │     │                    │    │
│  │  Your tools:        │     │  Targets:          │    │
│  │  nmap, burp, hydra  │     │  DVWA (:80/dvwa/)  │    │
│  │  ffuf, sqlmap       │     │  FTP (:21)         │    │
│  │                     │     │  SSH (:22)         │    │
│  │  Local targets:     │     │  Telnet (:23)      │    │
│  │  Juice Shop :3000   │     │  SMTP (:25)        │    │
│  │  Flask XXE  :5000   │     │  SMB (:139/445)    │    │
│  │  MobSF      :8000   │     │  MySQL (:3306)     │    │
│  └────────────────────┘     └───────────────────┘    │
└──────────────────────────────────────────────────────┘
```

---

## Ready? Start with [Challenge 9: Cookie Tampering](./01_lab_cookie_tampering.md) 🚀
