# 💼 Resume & Portfolio Strategy

> Without professional experience, your portfolio IS your experience. This document shows you exactly what to build, how to document it, and how to present yourself.

---

## 🎯 The Golden Rule
**If it's not public and documented, it doesn't exist.**

You've already done great lab work. But it's invisible to employers. Your #1 job for the next 6 months is making your skills VISIBLE.

---

## 📄 Resume Strategy

### Resume Structure (For Cybersecurity Transition)

```
1. Contact Info + Links (GitHub, LinkedIn, Blog, Portfolio Website)
2. Professional Summary (4 lines max)
3. Technical Skills (organized by category)
4. Cybersecurity Projects & Lab Experience (THIS IS YOUR MAIN SECTION)
5. Security Tools Proficiency
6. Certifications (even if "In Progress")
7. Software Development Experience (keep it brief, 3-4 bullets)
8. Education
```

### Your Professional Summary Template
```
Security-focused software developer transitioning to penetration testing and 
application security. [X months] of structured hands-on experience in web 
application testing, Active Directory attacks, and vulnerability assessment 
through self-built lab environments. Completed [X] PortSwigger labs, [X] 
Hack The Box machines, and built [X] security tools. Background in full-stack 
development (Angular, Spring Boot) provides unique code review and application 
security perspective that most pentesters lack.
```

### Critical Resume Rules for Your Situation

1. **Lead with cybersecurity, not dev experience.** Your dev job goes BELOW your security projects.
2. **Quantify everything.** "Completed 80+ PortSwigger labs" beats "practiced web application testing."
3. **Use action verbs specific to security.** Identified, exploited, compromised, escalated, documented, reported.
4. **Include your GitHub link prominently.** Make sure the repo is clean before applying.
5. **Customize for EVERY application.** Match keywords from the job description.
6. **One page maximum** for entry-level roles in India.
7. **No photo, no personal details** (age, marital status) unless the company specifically asks.

### What to Include vs. What to Cut

| Include | Cut |
|---------|-----|
| Specific HTB machines you solved | "Self-learner" (everyone says this) |
| PortSwigger lab count with categories | Generic "cybersecurity enthusiast" |
| AD attack chain paths you've practiced | List of every tool you've ever touched |
| Security tools you've BUILT | Soft skills without evidence |
| Pentest reports you've written | Certifications you plan to get "someday" |
| Bug bounty submissions (even if rejected) | Unrelated hobbies |
| Specific vulnerability types you can exploit | "Passionate about security" (show, don't tell) |

---

## 🛠️ Portfolio Projects to Build (Ordered by Impact)

### 🌟 Project 1: Professional Pentest Reports (Month 3-4)
**Resume Impact: ⭐⭐⭐⭐⭐**

Create 3-4 professional-grade penetration test reports.

**What to pentest:**
- Your own DVWA instance (at all security levels)
- OWASP Juice Shop
- A VulnHub machine
- Your AD lab (full internal pentest report)

**Report format:**
- Executive Summary (for non-technical readers)
- Scope and Methodology
- Findings (sorted by severity: Critical → High → Medium → Low → Informational)
- Each finding: Description → Impact → Steps to Reproduce → Evidence (screenshots) → Remediation
- Appendix with tools used

**Template:** Use TCM Security's PNPT report template or create your own.

### 🌟 Project 2: Security Tools on GitHub (Month 5-8)
**Resume Impact: ⭐⭐⭐⭐⭐**

Build 2-3 security tools that demonstrate coding + security skills.

**Tool Ideas (pick 2-3):**

| Tool | Tech | Difficulty | Why It Impresses |
|------|------|-----------|------------------|
| Recon automation framework | Python/Bash | Medium | Shows workflow automation |
| HTTP security header checker | Python | Easy | Practical, usable tool |
| Subdomain takeover scanner | Python | Medium | Shows cloud awareness |
| Pentest report generator (Markdown → PDF) | Python (Jinja2) | Medium | Solves real pentester problem |
| CORS misconfiguration detector | Python | Easy | Specific, useful |
| Custom Burp extension | Java/Python | Hard | Shows advanced Burp skills |
| API security scanner | Python | Medium | Shows API testing knowledge |
| Vulnerable web app (Spring Boot) | Java | Medium | Your dev skills + security |

**GitHub Repository Standards:**
- Clean README with: description, installation, usage, screenshots/GIFs
- Requirements.txt or proper dependency management
- Code comments explaining security logic
- License file (MIT is fine)
- CI/CD pipeline (GitHub Actions) if possible
- At least 1 star from yourself + share in communities for organic stars

### 🌟 Project 3: Technical Blog (Month 2+)
**Resume Impact: ⭐⭐⭐⭐**

Platform: Medium (for discoverability) or GitHub Pages (for control)

**Content Calendar:**

| Month | Blog Posts |
|-------|------------|
| 1-2 | "My Transition: From Software Developer to Pentester" (personal story) |
| 2-3 | HTB machine writeups (2-3 retired machines) |
| 3-4 | "How I Built My AD Lab and Compromised Domain Admin" |
| 4-5 | "Code Review for Security: A Developer's Guide" |
| 5-6 | Tool announcement posts (for your GitHub tools) |
| 6-7 | "Cloud Security Misconfigurations I Found in My Lab" |
| 7-8 | Bug bounty writeup (if you have a finding) |
| 9-10 | Technical deep dive on a vulnerability class |
| 11-12 | "One Year of Learning Cybersecurity: Honest Review" |

**Blog Rules:**
- Minimum 2 posts per month
- Target: 20+ posts by Month 12
- Always include code snippets, screenshots, commands
- Share every post on LinkedIn and Twitter

### 🌟 Project 4: Portfolio Website (Month 6)
**Resume Impact: ⭐⭐⭐⭐**

**What to include:**
- About Me (dev-to-security story)
- Skills & Tools (visual representation)
- Projects (links to GitHub tools)
- Pentest Reports (redacted samples)
- Blog (embedded or linked)
- Bug Bounty Profile (if any findings)
- Contact Info

**Tech stack:** Use your dev skills — simple HTML/CSS/JS or Next.js.

### 🌟 Project 5: Vulnerable Web Application (Month 5)
**Resume Impact: ⭐⭐⭐⭐⭐**

Build a deliberately vulnerable web app using Spring Boot (your strength!).

**Include these vulnerabilities:**
- SQL Injection (authentication bypass)
- IDOR (access control bypass)
- Mass assignment
- Broken authentication (weak JWT implementation)
- XSS (stored and reflected)
- File upload vulnerability
- API rate limiting bypass

**Then:** Write a full pentest report for YOUR OWN app.

**Why this is powerful:** It shows you can BOTH build applications AND break them. No other candidate can demonstrate this.

---

## 🔗 Online Presence Checklist

| Platform | Action | Target by Month 6 |
|----------|--------|-------------------|
| **GitHub** | Clean repos, READMEs, pinned projects, contribution graph | 5+ pinned repos, green contribution graph |
| **LinkedIn** | Headline: "Penetration Tester | Application Security | Bug Bounty Hunter" | 300+ connections, weekly posts |
| **Medium/Blog** | 2 posts/month, share in security communities | 12+ posts |
| **HackerOne/Bugcrowd** | Active profile, even if 0 bounties | Profile exists, reputation building |
| **Twitter/X** | Follow security researchers, share learnings | 100+ followers, daily engagement |
| **TryHackMe** | Public profile showing progress | Top 5% ranking |
| **HTB** | Public profile | Hacker rank or above |
| **Portfolio Website** | Professional site showcasing everything | Live by Month 6 |

---

## 📝 How to Describe Lab Experience as Professional Experience

You don't have professional experience. But you have LAB experience. Here's how to present it:

**❌ Wrong way:**
> "Practiced SQL injection on DVWA in my free time"

**✅ Right way:**
> "Conducted comprehensive web application penetration test on multi-tier application (DVWA) across three security configurations. Identified and exploited 10 vulnerability categories including SQL Injection, XSS, and authentication bypass. Documented findings in professional pentest report with risk ratings and remediation recommendations."

**❌ Wrong way:**
> "Set up Active Directory lab and hacked it"

**✅ Right way:**
> "Designed and executed internal penetration test on multi-machine Active Directory environment (Windows Server 2019 DC, 2 workstations). Achieved Domain Admin compromise through 3 distinct attack paths including Kerberoasting, LLMNR poisoning with credential relay, and DCSync attack. Mapped attack paths using BloodHound and documented findings with MITRE ATT&CK framework mapping."

**Key transformation principles:**
1. Use professional terminology ("conducted" not "practiced")
2. Include scope (how many machines, what technologies)
3. Quantify findings ("10 vulnerability categories")
4. Mention methodology and frameworks (OWASP, MITRE ATT&CK)
5. Emphasize deliverables ("documented in professional report")

---

## 📈 Portfolio Readiness Checklist

**By Month 6 (minimum for job applications):**
- [ ] GitHub: 3+ security projects with clean READMEs
- [ ] Blog: 10+ published posts
- [ ] Pentest Reports: 3 professional reports in portfolio
- [ ] Portfolio Website: Live and updated
- [ ] LinkedIn: 200+ connections, regular posting
- [ ] Resume: Customized v1 ready
- [ ] 1 certification earned (or scheduled)

**By Month 12 (job-ready):**
- [ ] GitHub: 5+ projects including custom tools
- [ ] Blog: 20+ posts
- [ ] Pentest Reports: 5+ reports including AD and web app
- [ ] Bug Bounty: At least 1 valid submission
- [ ] CTF: 3+ competitions participated
- [ ] Portfolio Website: Professional, up-to-date
- [ ] LinkedIn: 500+ connections
- [ ] Resume: Multiple versions for different role types

---

*Previous: [06 — Certification Guidance ←](./06_certification_guidance.md) | Next: [08 — Job Preparation Strategy →](./08_job_preparation_strategy.md)*
