# 🌐 Phase 2: Web Application Mastery (Months 4-7)

> **"AI will automate basic web scans. Humans who understand browser internals, auth systems, and multi-step logic chains will remain irreplaceable."**

---

## 🎯 Objective

Graduate from basic web vuln exploitation (XSS, SQLi, CSRF) to **advanced, research-grade web hacking** — the kind that wins bug bounties and discovers 0-days.

---

## 📅 Timeline: 4 Months

| Week | Focus | Deliverable |
|---|---|---|
| 1-2 | PortSwigger Practitioner Labs (all categories) | Complete 80%+ of practitioner-level labs |
| 3-4 | Advanced Authentication Attacks | OAuth/OIDC/SAML/JWT exploit writeups |
| 5-6 | API Hacking (REST, GraphQL, gRPC) | Custom API fuzzer tool |
| 7-8 | HTTP Request Smuggling & Desync | Working smuggling PoC |
| 9-10 | Web Cache Poisoning & Deception | Cache poisoning writeup |
| 11-12 | Race Conditions & Business Logic | Complex multi-step chain writeup |
| 13-14 | Browser Internals & Client-Side | Prototype pollution + XS-Leak PoC |
| 15-16 | Automation & Custom Tooling | Custom Burp extension + recon tool |

---

## 1️⃣ Advanced Injection Attacks

### Beyond Basic SQLi and XSS

| Attack | What to Learn |
|---|---|
| **Blind SQLi** | Time-based, boolean-based, out-of-band (DNS/HTTP) |
| **Second-order SQLi** | Injections that trigger later in different contexts |
| **NoSQL Injection** | MongoDB, CouchDB — operator injection, auth bypass |
| **LDAP Injection** | LDAP query manipulation for auth bypass |
| **Template Injection (SSTI)** | Jinja2, Twig, Freemarker, Pebble — to RCE |
| **Expression Language Injection** | Spring EL, OGNL, MVEL — direct code execution |
| **DOM XSS** | Source → sink tracing, DOM clobbering |
| **Mutation XSS (mXSS)** | Browser parser differentials for filter bypass |
| **XSS in Modern Frameworks** | React, Angular, Vue — dangerouslySetInnerHTML, bypass patterns |

### Practice Labs

- PortSwigger: All SQL injection labs (Apprentice → Expert)
- PortSwigger: All XSS labs (Apprentice → Expert)
- PortSwigger: SSTI labs
- HackTheBox: Web challenges tagged "injection"

---

## 2️⃣ Authentication & Session Attacks

### What to Master

| Topic | Attack Vectors |
|---|---|
| **OAuth 2.0** | Redirect URI manipulation, state parameter attacks, PKCE bypass, token theft via open redirect |
| **OpenID Connect** | ID token manipulation, confused deputy, SSRF via discovery endpoint |
| **SAML** | XML signature wrapping, assertion manipulation, XXE in SAML |
| **JWT** | None algorithm, key confusion (RS256→HS256), JWK injection, kid path traversal |
| **Session Management** | Fixation, cookie flags, session puzzling, concurrent session attacks |
| **MFA Bypass** | Race conditions, response manipulation, backup code brute-force, MFA fatigue |
| **Password Reset** | Token prediction, host header injection, dangling markup |

### Exercises

1. Exploit OAuth redirect_uri validation bypass on PortSwigger
2. Forge a JWT using the "none" algorithm attack
3. Perform RS256 to HS256 key confusion attack
4. Exploit SAML XML signature wrapping
5. Bypass MFA using race condition techniques

### Resources

- 📖 **OAuth 2 in Action** — Justin Richer
- 🎓 **PortSwigger: Authentication labs** (all levels)
- 🔗 **OWASP Testing Guide: Authentication**

---

## 3️⃣ API Hacking

### Attack Surface

| API Type | Attack Vectors |
|---|---|
| **REST** | BOLA/IDOR, mass assignment, rate limiting bypass, parameter pollution |
| **GraphQL** | Introspection, batching attacks, nested query DoS, field suggestion exploitation |
| **gRPC** | Protobuf manipulation, reflection abuse, metadata injection |
| **WebSocket** | Cross-site WebSocket hijacking, message manipulation, origin validation bypass |
| **Webhook** | SSRF via webhook, signature bypass, replay attacks |

### Exercises

1. Find and exploit BOLA in a REST API (OWASP crAPI)
2. Perform GraphQL introspection and extract hidden queries/mutations
3. Use batch queries in GraphQL to bypass rate limiting
4. Hijack a WebSocket connection cross-site
5. Build a custom API fuzzer that tests for BOLA, mass assignment, and injection

### Tools

| Tool | Purpose |
|---|---|
| **Burp Suite Pro** | Intercepting proxy, scanner |
| **Postman** | API testing and collection |
| **GraphQL Voyager** | Schema visualization |
| **InQL** | GraphQL vulnerability scanner (Burp extension) |
| **Arjun** | Hidden parameter discovery |
| **ParamSpider** | Parameter mining from web archives |

### Resources

- 📖 **Hacking APIs** — Corey Ball
- 🎓 **APIsec University** (free)
- 🧪 **OWASP crAPI** — Completely Ridiculous API (practice target)
- 🧪 **vAPI** — Vulnerable API for practice

---

## 4️⃣ HTTP Request Smuggling & Desync Attacks

### What to Master

| Technique | Details |
|---|---|
| **CL.TE Smuggling** | Content-Length vs Transfer-Encoding conflict |
| **TE.CL Smuggling** | Reverse of above |
| **TE.TE Smuggling** | Obfuscating Transfer-Encoding header |
| **HTTP/2 Downgrade** | H2 → H1.1 smuggling |
| **h2c Smuggling** | HTTP/2 cleartext upgrade abuse |
| **Client-Side Desync** | Browser-powered request smuggling |
| **Response Queue Poisoning** | Desynchronizing response queue |

### Exercises

1. Complete all PortSwigger HTTP request smuggling labs
2. Build a custom smuggling detection script
3. Exploit CL.TE to capture another user's request
4. Practice HTTP/2 downgrade smuggling

### Resources

- 🎓 **PortSwigger Research** — James Kettle's talks on HTTP desync
- 📖 **HTTP/2: A Deep Dive** — understanding the protocol
- 🎯 **PortSwigger: All smuggling labs**

---

## 5️⃣ Web Cache Poisoning & Deception

### Topics

| Attack | Details |
|---|---|
| **Cache Poisoning** | Unkeyed headers, fat GET requests, parameter cloaking |
| **Cache Deception** | Path confusion to cache sensitive responses |
| **CDN Bypass** | Origin exposure, cache key manipulation |
| **Web Cache Poisoning via Gadgets** | Combining unkeyed inputs with gadgets (JS includes, redirects) |

### Exercises

1. Complete all PortSwigger web cache poisoning labs
2. Identify unkeyed headers using Param Miner (Burp extension)
3. Exploit cache deception to steal another user's data

---

## 6️⃣ Race Conditions & Business Logic

### What to Master

| Topic | Details |
|---|---|
| **Time-of-check-to-time-of-use (TOCTOU)** | Classic race condition pattern |
| **Single-endpoint races** | Parallel requests to same endpoint |
| **Multi-endpoint races** | Coordinated requests across endpoints |
| **Limit overrun** | Bypassing quantity/balance limits |
| **Business Logic Flaws** | Multi-step workflow bypasses, state manipulation |
| **HTTP/2 Single-packet Attack** | Sending multiple requests in one TCP packet for timing precision |

### Exercises

1. Complete all PortSwigger race condition labs
2. Exploit a limit overrun using Burp Repeater's parallel send
3. Find a business logic flaw in a multi-step checkout flow
4. Use Turbo Intruder for high-precision race attacks

---

## 7️⃣ Browser Internals & Client-Side Attacks

### What to Master

| Topic | Details |
|---|---|
| **Same-Origin Policy** | Deep understanding — what it does and does NOT protect |
| **CORS** | Misconfiguration exploitation, null origin, credential inclusion |
| **CSP** | Bypass techniques — nonce reuse, base-uri, script gadgets, JSONP endpoints |
| **Prototype Pollution** | Client-side and server-side, gadget chains |
| **DOM Clobbering** | Overwriting DOM objects for XSS |
| **XS-Leaks** | Cross-site information leakage via timing, error, frame counting |
| **Clickjacking** | Advanced techniques, drag-and-drop, multi-step |
| **Dangling Markup** | Exfiltrating data via injected HTML |

### Exercises

1. Exploit CORS misconfiguration to steal sensitive data
2. Bypass CSP using a JSONP endpoint gadget
3. Perform prototype pollution → XSS chain
4. Implement an XS-Leak to detect a logged-in user

### Resources

- 🎓 **PortSwigger: All client-side labs**
- 📖 **The Tangled Web** — Michal Zalewski
- 🔗 **xsleaks.dev** — XS-Leaks wiki

---

## 8️⃣ Automation & Custom Tooling

### Build These

| Tool | What It Does |
|---|---|
| **Custom Burp Extension** | Automate a specific check (e.g., JWT analysis, param tampering) |
| **Recon Pipeline** | Subdomain enum → alive check → screenshot → vuln scan |
| **Parameter Fuzzer** | Discover hidden parameters and test for injection |
| **Auth Tester** | Automated BOLA/IDOR testing across endpoints |
| **JS Analyzer** | Extract endpoints, secrets, and sinks from JavaScript files |

### Tech Stack for Web Hacking Automation

```
Python: requests, beautifulsoup4, selenium, playwright
Burp Extensions: Jython + Burp Extender API
JavaScript: Puppeteer/Playwright for browser automation
Bash: Piping tools together (subfinder | httpx | nuclei)
```

---

## 📝 Phase 2 Completion Checklist

- [ ] Completed 80%+ of PortSwigger Practitioner-level labs
- [ ] Can exploit OAuth/SAML/JWT in realistic scenarios
- [ ] Can identify and exploit BOLA, mass assignment in APIs
- [ ] Built a working HTTP request smuggling exploit
- [ ] Exploited web cache poisoning and deception
- [ ] Found business logic flaws in multi-step workflows
- [ ] Understand SOP, CORS, CSP deeply and can bypass each
- [ ] Performed prototype pollution → impact chain
- [ ] Built a custom Burp extension
- [ ] Built a recon automation pipeline

---

## 🔗 Platforms for This Phase

| Platform | What to Do |
|---|---|
| **PortSwigger Academy** | ALL labs — Apprentice through Expert |
| **HackTheBox** | Web challenges + Medium web machines |
| **OWASP crAPI** | API hacking practice |
| **Damn Vulnerable GraphQL** | GraphQL exploitation |
| **PentesterLab** | Auth and web attack exercises |

---

**Next: `04_phase3_network_and_infra_attacks.md` →**
