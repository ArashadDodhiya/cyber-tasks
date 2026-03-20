# 📘 Week 1: Observe — See What Others Miss

> **Theme**: Train your eyes to see attack surfaces, hidden information, and system architecture everywhere.

---

## Day 1: The Invisible Web

**Goal**: Learn to see what browsers hide from you.

### Tasks
- [ ] Pick any 3 websites you use daily (e.g., Gmail, YouTube, Amazon)
- [ ] For each site, open **Developer Tools** (F12) and explore:
  - **Elements tab**: Find hidden form fields (`type="hidden"`)
  - **Network tab**: Watch requests as you click around — note API endpoints
  - **Console tab**: Check for JavaScript errors or debug messages
  - **Application tab**: List all cookies and localStorage items
- [ ] Write down: What surprised you? What data is being sent that you didn't expect?

### Hacker Thinking
> *"Every website is an iceberg. Users see the tip. Hackers see what's underwater."*

---

## Day 2: HTTP Under the Microscope

**Goal**: Understand what actually travels over the wire.

### Tasks
- [ ] Open **Burp Suite** (or browser DevTools Network tab)
- [ ] Visit a website and capture the HTTP request/response
- [ ] For one request, document:
  - Request method (GET/POST)
  - All headers (Host, User-Agent, Cookie, Authorization, etc.)
  - Request body (if POST)
  - Response status code
  - Response headers (Server, X-Powered-By, Set-Cookie, etc.)
  - Response body (first 20 lines)
- [ ] Identify: What information does the **server** leak about itself?

### Hacker Thinking
> *"HTTP headers are like a name tag at a conference — they tell you more than the person intended."*

---

## Day 3: Recon Like a Pro

**Goal**: Master passive reconnaissance techniques.

### Tasks
- [ ] Choose a target domain (your own site or a bug bounty program)
- [ ] Perform a 15-minute recon session gathering:
  - [ ] WHOIS information (use `whois` or online tools)
  - [ ] DNS records (`nslookup`, `dig`, or [dnsdumpster.com](https://dnsdumpster.com))
  - [ ] Subdomains (use [crt.sh](https://crt.sh) for certificate transparency)
  - [ ] Technology stack (use [Wappalyzer](https://www.wappalyzer.com/) or [BuiltWith](https://builtwith.com/))
  - [ ] `robots.txt` and `sitemap.xml`
- [ ] Compile everything into a mini recon report

### Hacker Thinking
> *"Hacking doesn't start with exploitation — it starts with information."*

---

## Day 4: Source Code Secrets

**Goal**: Train yourself to read page source for hidden clues.

### Tasks
- [ ] Visit 5 different websites and view their page source (Ctrl+U)
- [ ] Hunt for:
  - [ ] HTML comments (`<!-- -->`) — developers often leave notes/TODOs
  - [ ] Hidden form fields and their values
  - [ ] JavaScript files — especially inline scripts with API keys, endpoints, or tokens
  - [ ] Meta tags with version info or generator names
  - [ ] Links to admin panels, staging environments, or internal tools
- [ ] Document the most interesting finding from each site

### Hacker Thinking
> *"Comments in code are love letters from developers to hackers."*

---

## Day 5: Map the Attack Surface

**Goal**: Learn to systematically identify all entry points of an application.

### Tasks
- [ ] Pick one web application (Juice Shop or DVWA recommended)
- [ ] Create an **attack surface map** listing:
  - [ ] All visible input fields (search bars, login forms, comment boxes)
  - [ ] URL parameters (anything after `?` in the URL)
  - [ ] Cookies and session tokens
  - [ ] File upload features
  - [ ] API endpoints (found via Network tab)
  - [ ] Authentication mechanisms
  - [ ] Areas that handle user content (profiles, messages, reviews)
- [ ] Rate each entry point: **Low / Medium / High** risk

### Hacker Thinking
> *"An attacker only needs one door. A defender must guard them all."*

---

## Day 6: The Art of Google Dorking

**Goal**: Use search engines as hacking tools.

### Tasks
- [ ] Learn and practice these Google dork operators:
  ```
  site:example.com                    → Limit results to one domain
  filetype:pdf site:example.com       → Find specific file types
  intitle:"index of"                  → Find open directory listings
  inurl:admin                         → Find admin pages
  "powered by" site:example.com       → Identify technologies
  ext:sql | ext:env | ext:log         → Find sensitive file types
  "error" | "warning" site:example.com → Find error pages
  ```
- [ ] Try 5 different dork queries against a domain you own or a practice target
- [ ] Document: What sensitive information could you find?

### Hacker Thinking
> *"Google is the world's largest vulnerability scanner — most people just don't know how to use it."*

---

## Day 7: Week 1 Reflection & Recon Report

**Goal**: Consolidate what you've learned and build your first complete recon report.

### Tasks
- [ ] Choose one target (your own site, a CTF, or a bug bounty scope)
- [ ] Spend 30 minutes doing a full recon using ALL Day 1-6 techniques
- [ ] Create a structured report with:
  - **Target Overview** (domain, IP, tech stack)
  - **Attack Surface Map** (all entry points found)
  - **Information Leaks** (headers, comments, hidden fields)
  - **Interesting Findings** (anything unusual or potentially exploitable)
  - **Risk Assessment** (what seems most promising to test further)

### Reflection Questions
- [ ] What type of information was easiest to find?
- [ ] What surprised you most this week?
- [ ] How has your perception of websites changed?

### Hacker Thinking
> *"A good hacker spends 80% of their time on recon and 20% on exploitation."*

---

## ✅ Week 1 Skills Checklist

After this week, you should be comfortable with:

- [ ] Reading HTTP requests/responses
- [ ] Using browser DevTools for security analysis
- [ ] Performing passive reconnaissance
- [ ] Identifying hidden information in source code
- [ ] Mapping an application's attack surface
- [ ] Using Google dorks for information gathering
- [ ] Writing a structured recon report
