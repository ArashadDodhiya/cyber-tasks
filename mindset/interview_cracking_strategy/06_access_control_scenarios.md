# 🛡️ Access Control Issues — Interview Scenarios

> *"Access control is checked on every request. Find the one where the developer forgot."*

---

## Scenario 1: Broken Access Control Discovery (Medium)

**🎯 Interviewer:** "You have a regular user account. How do you systematically test for access control issues?"

### ✅ Model Answer:

"I use a methodical approach with multiple accounts:

**Setup:** Create accounts at different privilege levels:
- Regular user (Account A)
- Another regular user (Account B)
- Admin user (if possible)

**Step 1 — Map all endpoints and functions:**
Log every request as admin. Create a spreadsheet:

| Endpoint | Method | Admin Response | User A Response | No Auth |
|----------|--------|----------------|-----------------|---------|
| `/api/admin/users` | GET | 200 (user list) | ? | ? |
| `/api/users/1/delete` | DELETE | 200 | ? | ? |
| `/api/settings/global` | PUT | 200 | ? | ? |

**Step 2 — Replay admin requests as regular user:**
In Burp Suite, use the 'Authorize' extension:
1. Browse as admin (captures all requests)
2. Extension replays each request with regular user's session
3. Compares responses — flags differences

**Step 3 — Test specific patterns:**
```
Force browsing:    GET /admin, /dashboard, /internal, /debug
Method switching:  Can't GET? Try POST, PUT, PATCH, DELETE
Path manipulation: /api/v1/admin/users vs /api/v2/admin/users
Parameter adding:  /api/data?admin=true, ?role=admin, ?debug=1
Header tricks:     X-Original-URL: /admin, X-Rewrite-URL: /admin
```

**Step 4 — Test horizontal access:**
As User A, access User B's resources. As User B, access User A's. Check both directions — sometimes access control is asymmetric."

### 🔥 Follow-ups:

**Q: "What's the difference between authentication and authorization flaws?"**

> "**Authentication** = 'Who are you?' — Proving identity. Flaws: weak passwords, broken session management, JWT issues.
> 
> **Authorization** = 'What are you allowed to do?' — Permission enforcement. Flaws: accessing admin pages as user, viewing other users' data.
> 
> An app can have perfect authentication (you definitely proved who you are) but broken authorization (it doesn't check what you're allowed to do). These are independent systems.
> 
> Example: You log in successfully (authentication works), but then access `/admin/delete-user/5` (authorization missing) — authenticated but not authorized."

---

## Scenario 2: Forced Browsing & Hidden Endpoints (Medium)

**🎯 Interviewer:** "The admin panel has no visible link in the UI for regular users. Is it secure?"

### ✅ Model Answer:

"Absolutely not. Security through obscurity is not security. I'd test:

**1. Common admin paths:**
```
/admin, /administrator, /admin-panel, /admin.php
/dashboard, /portal, /internal
/manage, /management, /console
/api/admin, /api/v1/admin
/_admin, /wp-admin (WordPress)
```

**2. Directory bruteforce:**
```
gobuster dir -u https://target.com -w /usr/share/wordlists/common.txt
ffuf -u https://target.com/FUZZ -w admin-wordlist.txt
```

**3. JavaScript source analysis:**
```javascript
// Often hidden endpoints are referenced in JS
fetch('/api/internal/users')     // found in bundle.js
axios.get('/admin/dashboard')    // found in app.js
```
I search all JS files for: `api/`, `admin`, `internal`, `fetch(`, `axios`, route definitions.

**4. Robots.txt and sitemap:**
```
# robots.txt often reveals hidden paths
Disallow: /admin
Disallow: /internal-tools
Disallow: /debug
```
What they're trying to hide from crawlers is exactly what I want to find.

**5. API documentation:**
- `/swagger`, `/api-docs`, `/swagger.json`, `/openapi.json`
- Swagger UI often lists ALL endpoints, including admin ones

**The question isn't 'can I find the URL?' — it's 'does the server check authorization when I access it?'**"

---

## Scenario 3: Privilege Escalation via Parameter Manipulation (Hard)

**🎯 Interviewer:** "During user registration, you see the request includes a `role` field. How do you exploit this?"

### ✅ Model Answer:

"This is a mass assignment / privilege escalation scenario.

**Step 1 — Observe the registration request:**
```json
POST /api/register
{
  "username": "testuser",
  "email": "test@test.com",
  "password": "SecurePass123"
}
```

**Step 2 — Check what fields the API accepts:**
Look at the user profile response for extra fields:
```json
GET /api/profile
{
  "username": "testuser",
  "email": "test@test.com",
  "role": "user",
  "is_admin": false,
  "plan": "free",
  "org_id": 5
}
```

**Step 3 — Add those fields to registration:**
```json
POST /api/register
{
  "username": "hacker",
  "email": "hack@test.com",
  "password": "SecurePass123",
  "role": "admin",
  "is_admin": true,
  "plan": "enterprise"
}
```

**Step 4 — Also test on profile update:**
```json
PUT /api/profile
{
  "username": "testuser",
  "role": "admin"
}
```

**Step 5 — Try different field names:**
```json
{"admin": true}
{"isAdmin": true}
{"is_superuser": true}
{"user_type": "administrator"}
{"group": "admins"}
{"permissions": ["admin", "super"]}
```

**Why this works:** Frameworks like Rails, Django, and Express can automatically bind all request parameters to model attributes. If the developer didn't explicitly whitelist allowed fields, the server accepts and saves whatever you send. This is 'mass assignment' — and it's how many real privilege escalations happen."

### 🔥 Follow-ups:

**Q: "The server returns 200 OK but your role doesn't actually change. What next?"**

> "The server might accept the field without applying it. But I'd:
> 1. Log out and log back in — maybe the role applies on next session
> 2. Check different endpoints — maybe one API applies it while another doesn't
> 3. Try via different content types — JSON, XML, form-urlencoded
> 4. Check if the field is applied asynchronously (delayed processing)
> 5. Try during password change or profile update instead of registration
> 6. Look for a different parameter name — `type`, `level`, `access`, `group`"

---

## Scenario 4: HTTP Method-Based Access Control (Medium-Hard)

**🎯 Interviewer:** "You can GET `/api/users` but only admins should be able to DELETE. How do you test?"

### ✅ Model Answer:

"I'd test every HTTP method on every endpoint:

```
GET    /api/users/5  → 200 (allowed, I can read)
POST   /api/users    → 403 (can't create — good)
PUT    /api/users/5  → ?   (can I modify?)
PATCH  /api/users/5  → ?   (can I partially modify?)
DELETE /api/users/5  → ?   (can I delete?)
```

**Common access control mistakes with methods:**

1. **Only checking on specific methods:**
   - Developer added auth check on DELETE but forgot PUT
   - `PUT /api/users/5 {"active": false}` effectively disables the account

2. **HEAD and OPTIONS leak info:**
   ```
   HEAD /api/admin/users → 200 OK (reveals the endpoint exists)
   OPTIONS /api/admin/users → reveals allowed methods
   ```

3. **Method override headers:**
   ```
   POST /api/users/5 HTTP/1.1
   X-HTTP-Method-Override: DELETE
   ```
   Some frameworks support method override via headers. The server sees POST (which passes the check) but processes it as DELETE.
   
   Also try: `X-Method-Override`, `X-HTTP-Method`, `_method` parameter.

4. **PATCH vs PUT behavior:**
   ```
   PUT /api/users/5       → 403 (blocked)
   PATCH /api/users/5     → 200 (not blocked!)
   ```
   Developers often forget PATCH when they protect PUT.

**Systematic test matrix:**
For each endpoint, test each method with each role. The `?` cells are your targets."

---

## Scenario 5: Multi-Tenant Access Control (Hard)

**🎯 Interviewer:** "The application is multi-tenant — Company A and Company B use the same platform. How do you test for cross-tenant access?"

### ✅ Model Answer:

"Cross-tenant vulnerabilities are critical — this is data breach territory.

**Setup:** Create accounts in two different organizations/tenants.

**Test 1 — Resource access across tenants:**
```
Tenant A: GET /api/projects/100  → 200 (Tenant A's project)
As Tenant B: GET /api/projects/100  → 200? or 403?
```

**Test 2 — Invitation/sharing across tenants:**
```
Can Tenant A invite a user from Tenant B?
Can Tenant A share a resource with Tenant B?
Does the API validate that the target belongs to the same tenant?
```

**Test 3 — Tenant identifier manipulation:**
```
Headers:  X-Tenant-ID: tenant_a → change to tenant_b
Cookies:  tenant=company_a → change to company_b
Subdomain: companya.app.com → try companyb.app.com with companya's session
API path:  /api/tenants/123/data → change to /api/tenants/124/data
```

**Test 4 — Search/listing across tenants:**
```
GET /api/users?search=admin
→ Does this return users from other tenants?

GET /api/projects?filter=all
→ Does 'all' include other tenants' projects?
```

**Test 5 — Shared resources:**
```
File uploads: /uploads/tenant_a/file.pdf → /uploads/tenant_b/file.pdf
Shared S3 buckets: predictable paths between tenants
Shared databases: SQL injection might access all tenants' data
```

**Why this matters:** A single cross-tenant IDOR could expose ALL customers' data. In SaaS products, this is the highest-severity finding possible — it's a multi-customer data breach."

---

## Quick Recall — Access Control Cheat Sheet

```
TESTING APPROACH:
  1. Map all endpoints as highest-privilege user
  2. Replay each request as lower-privilege user
  3. Test horizontal (user→user) and vertical (user→admin)
  4. Use Burp 'Authorize' extension for automation

FORCE BROWSING:
  /admin, /dashboard, /internal, /debug, /console
  /swagger, /api-docs, /graphql
  Check: robots.txt, sitemap.xml, JavaScript files

METHOD-BASED:
  Test ALL methods: GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD
  Method override: X-HTTP-Method-Override, _method param

PARAMETER MANIPULATION:
  Add: role=admin, is_admin=true, user_type=administrator
  Mass assignment: send extra fields the API might process

PATH TRICKS:
  /admin → 403?  Try:
  /Admin, /ADMIN, /admin/, /admin/., /admin..;/
  /./admin, //admin, /admin%20
  X-Original-URL: /admin
  
MULTI-TENANT:
  Cross-tenant resource access
  Tenant ID in headers/cookies/paths
  Search leaking across tenants
  Shared storage paths
```
