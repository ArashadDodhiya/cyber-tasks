# 👑 EXPERT Challenges (Success Ratio: < 5%) - Bug Bounty & OSWE Level

> Since you have already mastered advanced WAF bypasses, blind SQLi, mobile pinning bypasses, and thick clients, this tier focuses entirely on the **PortSwigger BSCP, OSWE, and modern Bug Bounty platforms (HackerOne/Bugcrowd)**.
> This level tests **Business Logic Flaws, Chain Attacks, and Modern Framework Exploitation**.

---

## Challenge 44: HTTP Request Smuggling (CL.TE & TE.CL)

### What They Test
Can you desynchronize front-end load balancers/reverse proxies and backend servers to bypass security controls or poison caches?

### Methodology
1. **Identify Proxy Architecture:** Is there a front-end server (like HAProxy/Nginx) routing to a backend?
2. **Test CL.TE (Content-Length on Front, Transfer-Encoding on Back):**
   ```http
   POST / HTTP/1.1
   Host: target.com
   Content-Length: 13
   Transfer-Encoding: chunked

   0

   SMUGGLED
   ```
3. **Test TE.CL (Transfer-Encoding on Front, Content-Length on Back):**
   ```http
   POST / HTTP/1.1
   Host: target.com
   Content-Length: 3
   Transfer-Encoding: chunked

   8
   SMUGGLED
   0

   ```
4. **Impact:** Smuggle a request that steals other users' session cookies via a malicious Host header leading to Web Cache Poisoning, or bypass WAF rule sets.

---

## Challenge 45: Advanced OAuth 2.0 & OIDC Misconfigurations

### What They Test
Can you exploit flaws in the Single Sign-On (SSO) delegation process to achieve full account takeover?

### Methodology
1. **Implicit Grant Flow Misuse:** Check if the application extracts tokens from the URL fragment (`#access_token=`) via JavaScript. Is the token validated against the intended client?
2. **CSRF in Authorization (`state` parameter):** If the `state` parameter is missing or predictable, initiate a login on your account, capture the callback URL, and force a victim to click it (they are logged into *your* account, revealing their actions/files to you).
3. **Flawed Redirect URIs (`redirect_uri`):**
   - Use path traversal: `redirect_uri=https://client.com/callback/../open_redirect`
   - Use parameter pollution: `redirect_uri=https://client.com/callback&redirect_uri=https://evil.com`
   - Subdomain takeover on trusted callback domains.
4. **OIDC OpenID Connect SSRF:** Exploit the dynamic client registration or `jwks_uri` (JSON Web Key Set) pointing to an attacker-controlled server to forge valid signatures.

---

## Challenge 46: Advanced Race Conditions (Time-of-Check to Time-of-Use)

### What They Test
Can you exploit concurrency issues by sending simultaneous requests to bypass limits (like coupon usage, financial transfers, or username registration)?

### Methodology
1. **Identify limits:** Look for features like "Use this coupon once," "Transfer $100," or "Upvote once."
2. **Turbo Intruder / HTTP/2 Single-Packet Attack:**
   - Standard Burp Intruder is too slow.
   - Use **Burp Turbo Intruder** or HTTP/2 multi-request pipeline.
   - Place 20-30 identical requests in a single TCP packet so the server processes them at the exact same millisecond.
3. **Impact:** Applying the same $10 discount 20 times to make an order free, or transferring money you don't have.

---

## Challenge 47: Web Cache Poisoning & Deception

### What They Test
Can you manipulate how edge nodes (Cloudflare, Akamai, Fastly) cache responses to serve malicious content to other users?

### Methodology
1. **Find Unkeyed Inputs:** Use Param Miner (Burp extension) to find HTTP headers (like `X-Forwarded-Host`, `X-Original-URL`, `X-Host`) that affect the application's response but are NOT part of the cache key.
2. **Poisoning (XSS):**
   ```http
   GET /home HTTP/1.1
   Host: target.com
   X-Forwarded-Host: evil.com
   ```
   If the server responds with `<script src="https://evil.com/script.js"></script>` AND caches the response, anyone visiting `/home` gets XSS.
3. **Web Cache Deception (Stealing Data):**
   - Request `GET /profile/my_api_key.css` (Target ignores `.css` and renders the profile data, but Cloudflare caches it because of the `.css` extension).
   - The victim visits the link, their API key is cached globally.
   - You download the cached `.css` file to steal their key.

---

## Challenge 48: Server-Side Prototype Pollution (Node.js)

### What They Test
Can you manipulate JavaScript object prototypes to achieve Remote Code Execution (RCE) or Authentication Bypass?

### Methodology
1. **Identify the Sink:** Look for recursive merges, object cloning, or uncontrolled JSON parsing.
   ```json
   {"__proto__": {"isAdmin": true}}
   ```
2. **Check Context:** If `isAdmin` is undefined for a user object, JS climbs the prototype chain, finds your injected `__proto__`, and grants admin access.
3. **RCE via Prototype Pollution:**
   - Find gadgets like `child_process.spawn()` or `fork()`.
   - Pollute environment variables: `{"__proto__": {"NODE_OPTIONS": "--require /proc/self/environ"}}`.

---

## Challenge 49: GraphQL Introspection & Batching Attacks

### What They Test
Can you map and exploit hidden data structures in GraphQL endpoints?

### Methodology
1. **Introspection:** Send an introspection query to dump the entire database schema (objects, fields, mutations).
2. **InQL / Clairvoyance:** Use tools to reconstruct the schema even if introspection is disabled by bruteforcing field names based on error messages (e.g., "Did you mean X?").
3. **BOLA (Insecure Direct Object Reference) in GraphQL:**
   ```graphql
   query {
     user(id: "1002") {
       email
       password_hash
     }
   }
   ```
4. **Query Batching for Brute Force:** Bypass rate limits by sending thousands of login attempts in a **single** HTTP request.
   ```json
   [
     {"query": "mutation { login(user:\"admin\", pass:\"123\") { token } }"},
     {"query": "mutation { login(user:\"admin\", pass:\"password\") { token } }"}
   ]
   ```

---

## Challenge 50: Advanced Server-Side Request Forgery (SSRF)

### What They Test
Can you bypass SSRF filters and extract sensitive cloud metadata?

### Methodology
1. **Filter Evasion Strategies:**
   - IP obfuscation: `http://2852039166` (Decimal for 169.254.169.254)
   - Hex encoding: `http://0xA9FEA9FE`
   - DNS Rebinding: Attacker DNS server returns a benign IP on the first check, but `169.254.169.254` (or `127.0.0.1`) on the second check.
2. **Cloud Exfiltration:**
   - Target AWS: `http://169.254.169.254/latest/meta-data/iam/security-credentials/`
   - Target GCP: `http://metadata.google.internal/computeMetadata/v1/` (Needs `Metadata-Flavor: Google` header - try to smuggle headers if possible or look for endpoints that mirror headers).
3. **Blind SSRF:**
   - Use Burp Collaborator / interactsh.
   - Escalate to RCE by hitting internal services (e.g., hitting an internal Redis instance without auth using `gopher://` or `dict://` protocols).

---

## Challenge 51: Insecure Deserialization (Java/PHP/C#)

### What They Test
Can you forge serialized objects to execute arbitrary code?

### Methodology
1. **Identify Serialized Data:**
   - Java: Starts with `rO0` (Base64) or `aced 0005` (Hex).
   - PHP: `O:4:"User":2:{s:8:"username"...}`
   - Python: Pickle data (often base64 encoded, ends with `.`).
2. **Generate Payloads:**
   - Java: Use **ysoserial** to generate common gadget chains (`CommonsCollections`, `Spring1`).
   - PHP: Use **phpggc**.
3. **Exploitation:** Find a class/gadget in the application's dependencies that executes dangerous operations (like `Runtime.exec()`) during the deserialization (`readObject()` or `__wakeup()`) phase.

---

## 🎯 Next Steps for Your Training
To practice these specific attacks, rely entirely on the **PortSwigger Web Security Academy**. Their "Apprentice" level corresponds to Easy/Moderate, but their **"Practitioner"** and **"Expert"** labs are exactly designed to train you for these OSWE/BSCP level scenarios.
