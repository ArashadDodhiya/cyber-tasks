# 🎯 Phase 8: Bug Bounty & Real-World Application (Month 28+)

> **"Labs teach you to hack. Bug bounty teaches you to THINK like a hacker in the real world."**

---

## 🎯 Objective

Transition from lab environments to real-world targets. Find real vulnerabilities, earn bounties, build reputation, and develop a research mindset.

---

## 📅 Timeline: Ongoing (Lifelong)

| Phase | Duration | Focus |
|---|---|---|
| Warmup | Month 1-2 | VDP submissions, methodology refinement |
| Active Hunting | Month 3-6 | Consistent hunting on paid programs |
| Specialization | Month 6-12 | Deep focus on one area |
| Research | Year 2+ | Novel techniques, CVEs, conference talks |

---

## 1️⃣ Getting Started with Bug Bounty

### Platforms

| Platform | Best For |
|---|---|
| **HackerOne** | Largest platform, good for beginners (many VDPs) |
| **Bugcrowd** | Good programs, researcher-friendly |
| **Intigriti** | European programs, growing platform |
| **YesWeHack** | European, good programs |
| **Synack** | Invite-only, higher payouts |

### Start Here (First Month)

```
1. Sign up on HackerOne and Bugcrowd
2. Start with VDPs (Vulnerability Disclosure Programs) — no bounty, less competition
3. Read program scope CAREFULLY — understand what's in/out
4. Start with a SINGLE target — depth over breadth
5. Focus on one vulnerability class you're good at
6. Submit your first report — even if it's low severity
```

### Bug Bounty Methodology

```
Phase 1: Reconnaissance (40% of time)
├── Subdomain enumeration (subfinder, amass, assetfinder)
├── Port scanning (nmap, masscan)
├── Technology fingerprinting (Wappalyzer, whatweb)
├── Content discovery (ffuf, feroxbuster)
├── JavaScript analysis (LinkFinder, JSParser)
├── Wayback Machine / GAU for historical endpoints
├── GitHub/GitLab dorking for secrets
├── Google dorking for exposed admin panels
└── Shodan/Censys for exposed services

Phase 2: Mapping & Analysis (20% of time)
├── Map all functionality
├── Identify authentication mechanisms
├── Map API endpoints
├── Identify user roles and access levels
├── Note file upload, search, and other interesting features
└── Create a target-specific attack tree

Phase 3: Vulnerability Discovery (30% of time)
├── Test for IDOR/BOLA on every endpoint
├── Test authentication and authorization
├── Test for injection (SQLi, XSS, SSTI, etc.)
├── Test for SSRF on URL-fetching features
├── Test business logic in multi-step flows
├── Test race conditions on sensitive operations
├── Test for information disclosure
└── Test API-specific vulnerabilities

Phase 4: Exploitation & Reporting (10% of time)
├── Develop clean proof of concept
├── Demonstrate maximum impact
├── Write clear, professional report
└── Follow up and respond to triage
```

---

## 2️⃣ Recon Automation

### Build Your Recon Pipeline

```bash
# Example automated recon pipeline

# 1. Subdomain Enumeration
subfinder -d target.com -o subs_subfinder.txt
amass enum -d target.com -o subs_amass.txt
sort -u subs_*.txt > all_subs.txt

# 2. Alive Check
httpx -l all_subs.txt -o alive.txt -status-code -title -tech-detect

# 3. Port Scanning
naabu -list all_subs.txt -top-ports 1000 -o ports.txt

# 4. Screenshot
gowitness file -f alive.txt

# 5. Content Discovery
cat alive.txt | while read url; do
  ffuf -u "$url/FUZZ" -w wordlist.txt -mc 200,301,302,403 -o "ffuf_$(echo $url | md5sum | cut -d' ' -f1).json"
done

# 6. JavaScript Analysis
cat alive.txt | subjs | sort -u > js_files.txt
cat js_files.txt | while read js; do
  python3 LinkFinder.py -i "$js" -o cli
done

# 7. Vulnerability Scanning
nuclei -list alive.txt -t nuclei-templates/ -severity critical,high -o nuclei_results.txt

# 8. Parameter Discovery
cat alive.txt | paramspider --subs
```

### Essential Recon Tools

| Tool | Purpose |
|---|---|
| **subfinder** | Passive subdomain enumeration |
| **amass** | Comprehensive subdomain discovery |
| **httpx** | HTTP probing and technology detection |
| **ffuf** | Fast web fuzzer / content discovery |
| **nuclei** | Template-based vulnerability scanner |
| **naabu** | Port scanner |
| **gowitness** | Web screenshot tool |
| **gau** | Get All URLs from Wayback Machine |
| **ParamSpider** | Parameter mining |
| **LinkFinder** | JavaScript endpoint extraction |

---

## 3️⃣ Writing Great Reports

### Report Template

```markdown
# [Vulnerability Type] in [Feature/Endpoint]

## Summary
One sentence describing the vulnerability and its impact.

## Severity
[Critical/High/Medium/Low] — based on CVSS or platform guidelines

## Steps to Reproduce
1. Navigate to https://target.com/...
2. Intercept the request with Burp Suite
3. Modify parameter X to Y
4. Observe the response showing Z

## Proof of Concept
[Screenshot/Video/HTTP request-response]

## Impact
Describe what an attacker can do:
- Access other users' data (IDOR)
- Execute arbitrary code (RCE)
- Steal session tokens (XSS)

## Remediation
Suggested fix for the development team.

## References
- OWASP link
- CWE reference
```

### Report Tips

| ✅ Do | ❌ Don't |
|---|---|
| Clear, numbered reproduction steps | Vague descriptions |
| Show maximum impact | Under-sell the finding |
| Include HTTP requests/responses | Only include screenshots |
| Suggest remediation | Just point out the problem |
| Be professional and respectful | Be condescending or rude |
| Follow up promptly | Disappear after submission |

---

## 4️⃣ Specialization Paths in Bug Bounty

### Pick ONE and go DEEP

| Specialization | What to Focus On | High-Value Bugs |
|---|---|---|
| **IDOR/Access Control** | Authorization logic, API endpoints | Account takeover, data access |
| **Authentication** | OAuth, SAML, JWT, password reset | Account takeover |
| **SSRF** | URL fetching features, PDF generation | Internal access, cloud metadata |
| **Injection** | SQLi, SSTI, XSS in complex contexts | RCE, data exfiltration |
| **Business Logic** | Multi-step workflows, pricing, limits | Financial impact |
| **Mobile** | Android/iOS app testing, API analysis | App-specific vulnerabilities |

---

## 5️⃣ Building Reputation & Research

### Publishing Research

| Activity | Where |
|---|---|
| **Blog** | Personal blog (GitHub Pages, Hugo, Ghost) |
| **Twitter/X** | Share findings, techniques, tools |
| **Conference Talks** | BSides, DEF CON, Black Hat, local meetups |
| **CVEs** | Responsible disclosure → CVE assignment |
| **Tools** | Open-source offensive tools on GitHub |
| **Writeups** | Detailed writeups of disclosed bugs |

### Career Progression from Bug Bounty

```
VDP Submissions → Paid Bug Bounty → Consistent Findings
    → Specialization → Top Researcher on Platform
        → Invited Programs → Consulting → Full-time Red Team
            → Independent Research → Conference Speaker
                → Security Startup / Advisory Roles
```

---

## 📝 Phase 8 Completion Checklist

- [ ] Signed up on HackerOne and Bugcrowd
- [ ] Submitted first valid vulnerability to a VDP
- [ ] Built an automated recon pipeline
- [ ] Received first bounty payment
- [ ] Written 5+ professional bug reports
- [ ] Chosen a specialization area
- [ ] Published first technical blog post
- [ ] Found a bug using a novel technique or chain
- [ ] Built and shared an open-source tool
- [ ] Contributed to the security community

---

**Next: `10_certifications_roadmap.md` →**
