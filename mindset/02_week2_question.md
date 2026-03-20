# 📗 Week 2: Question — Develop the "But What If..." Instinct

> **Theme**: Train your brain to challenge every assumption, probe every boundary, and ask dangerous questions.

---

## Day 8: Break the Login

**Goal**: Think of every possible way to attack an authentication system.

### Tasks
- [ ] Open DVWA or Juice Shop login page
- [ ] WITHOUT touching any tools, brainstorm at least 10 attack ideas:
  - What if I enter SQL in the username field?
  - What if I send an empty password?
  - What if I change the POST to GET?
  - What if I modify the session cookie?
  - What if I brute-force with common passwords?
  - What if there's a default admin account?
  - What if I bypass the login page entirely by going to `/dashboard`?
  - What if I register with `admin@admin.com`?
  - What if I intercept and modify the server's response?
  - What if the "forgot password" flow is weaker?
- [ ] Now try 5 of your ideas — document which ones worked and why

### Hacker Thinking
> *"Before you touch a tool, think with your brain. The best hackers plan their attacks on paper first."*

---

## Day 9: Input Boundary Testing

**Goal**: Understand that every input has boundaries — and boundaries can be broken.

### Tasks
- [ ] Find 3 different input fields in any web app (search, comment, profile name)
- [ ] For each, test these boundary conditions:
  - [ ] **Empty input** — Does it accept nothing? What error appears?
  - [ ] **Very long input** (1000+ characters) — Does it truncate? Crash?
  - [ ] **Special characters** — `< > ' " ; & | \ / { } ( )`
  - [ ] **Unicode** — Emoji, RTL characters, null bytes (`%00`)
  - [ ] **Negative numbers** (for numeric fields)
  - [ ] **HTML tags** — `<b>test</b>` or `<img src=x>`
  - [ ] **Script tags** — `<script>alert(1)</script>`
- [ ] Document: Which inputs caused unexpected behavior?

### Hacker Thinking
> *"A developer tests if things work. A hacker tests if things break."*

---

## Day 10: Trust Boundary Analysis

**Goal**: Identify where applications place trust — and whether that trust is misplaced.

### Tasks
- [ ] Analyze one web application and identify its trust boundaries:
  - [ ] **Client vs Server**: What validation happens only on the client side?
    - Disable JavaScript — does the form still validate?
    - Try submitting data directly via Burp/curl, bypassing the browser
  - [ ] **Cookies**: Can you modify cookie values? What happens?
  - [ ] **Hidden fields**: Change hidden field values and resubmit
  - [ ] **URL parameters**: Can you access other users' data by changing IDs?
  - [ ] **File types**: If it accepts images, what if you upload `.php` or `.html`?
- [ ] Create a "Trust Map" — diagram showing where the app trusts user input

### Hacker Thinking
> *"Security breaks down at trust boundaries — where one component trusts another without verification."*

---

## Day 11: The IDOR Mindset

**Goal**: Develop the instinct to check if access controls actually exist.

### Tasks
- [ ] In Juice Shop or DVWA:
  - [ ] Log in as a regular user
  - [ ] Find any URL or API that contains an ID (e.g., `/api/users/1`, `/order/42`)
  - [ ] Change the ID number — can you see other users' data?
  - [ ] Try accessing admin endpoints without an admin account
  - [ ] Check if API responses contain more data than the UI shows
- [ ] For each test, document:
  - What you tried
  - What happened
  - Whether proper access control was in place

### Practice Exercise
```
# Example IDOR test flow:
1. Login as user A → Go to your profile → Note the URL
   URL: /api/users/5
   
2. Change 5 to 1 → Can you see user 1's data?
   
3. Try /api/users/1, /api/users/2, /api/users/3...
   
4. Check: Does the server verify YOU are allowed to see this data?
```

### Hacker Thinking
> *"Just because a button is hidden doesn't mean the endpoint is protected."*

---

## Day 12: Parameter Tampering Lab

**Goal**: Learn that URL parameters and form data are attacker-controlled.

### Tasks
- [ ] Find a web app feature that uses URL parameters (price, quantity, role, discount)
- [ ] Practice these tampering techniques:
  - [ ] **Price manipulation**: Change `price=100` to `price=1` or `price=-100`
  - [ ] **Role escalation**: Change `role=user` to `role=admin`
  - [ ] **Adding parameters**: What if you add `&admin=true` or `&debug=1`?
  - [ ] **Removing parameters**: What if you remove required fields?
  - [ ] **Changing HTTP method**: Send a GET instead of POST (or vice versa)
- [ ] Use Burp Suite Repeater to modify and resend requests

### Hacker Thinking
> *"Anything the client sends to the server is a suggestion, not a command. The server should verify — but often doesn't."*

---

## Day 13: Error Message Mining

**Goal**: Extract valuable information from error messages.

### Tasks
- [ ] Intentionally trigger errors in a web application:
  - [ ] Add a quote `'` to URL parameters to trigger SQL errors
  - [ ] Request a non-existent page to see the 404 page
  - [ ] Submit invalid data types (text in number fields)
  - [ ] Send malformed requests via Burp Suite
  - [ ] Try accessing forbidden directories
- [ ] For each error, document:
  - [ ] Does it reveal the technology stack? (e.g., Apache, PHP, MySQL)
  - [ ] Does it show file paths? (e.g., `/var/www/html/index.php`)
  - [ ] Does it show database information? (table names, query structure)
  - [ ] Does it show framework versions?
  - [ ] Can you use this info to craft a more targeted attack?

### Hacker Thinking
> *"Error messages are the server's confession. Listen carefully."*

---

## Day 14: Week 2 Reflection & Attack Planning

**Goal**: Combine questioning techniques into a structured attack methodology.

### Tasks
- [ ] Choose one target application
- [ ] Spend 30 minutes creating a **structured attack plan**:
  1. **Enumerate all inputs** (forms, URLs, cookies, headers, APIs)
  2. **Identify trust boundaries** (client-side vs server-side validation)
  3. **List assumptions** the developer made
  4. **Prioritize attack vectors** (which assumptions are most likely wrong?)
  5. **Plan specific tests** for top 5 vectors
- [ ] Execute your top 3 planned attacks
- [ ] Document results and lessons learned

### Reflection Questions
- [ ] When you use a web app now, do you automatically think about attack vectors?
- [ ] What's the most common mistake you've found developers making?
- [ ] Which "what if..." question led to your best finding?

### Hacker Thinking
> *"A plan doesn't guarantee success, but it guarantees that you won't miss the obvious."*

---

## ✅ Week 2 Skills Checklist

After this week, you should be comfortable with:

- [ ] Brainstorming multiple attack vectors before testing
- [ ] Testing input boundaries systematically
- [ ] Identifying and testing trust boundaries
- [ ] Finding and exploiting IDOR vulnerabilities
- [ ] Parameter tampering via URL and Burp Suite
- [ ] Extracting information from error messages
- [ ] Creating structured attack plans
