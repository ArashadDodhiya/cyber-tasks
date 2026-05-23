# 🎓 Certification Guidance

> Certifications are a necessary evil in India's job market. This guide tells you exactly which ones to get, in what order, and which to avoid — with honest cost analysis in INR.

---

## ⚠️ The Certification Reality in India

**What HR departments think:** "No certification = not qualified"
**What hiring managers think:** "Show me what you can do"
**What actually happens:** Your resume gets filtered out by HR before the hiring manager ever sees it.

**Solution:** Get the minimum certification needed to pass HR filters. Then let your skills win the technical interview.

---

## 🔴 Recommended Certification Sequence

### CERT 1: eJPT v2 (eLearnSecurity Junior Penetration Tester)
**Get this: Month 5-6**

| Detail | Info |
|--------|------|
| Cost | ~₹28,000-35,000 (INE Fundamentals subscription + exam voucher) |
| Difficulty | Beginner-Intermediate (you can pass this with your current trajectory) |
| Exam Format | Practical — real pentest on lab environment (no MCQs!) |
| Duration | 48 hours to complete exam |
| Validity | 3 years |
| Prep Time | 4-6 weeks (with your background, less) |
| Industry Value | Respected as a practical entry-level cert. Better than CEH. |

**Why eJPT first:**
- It's PRACTICAL — you actually hack machines in the exam
- Much cheaper than OSCP
- Proves hands-on skills, not multiple-choice memorization
- Well-recognized in Indian security companies
- Your lab experience already covers 70% of the syllabus
- Perfect stepping stone to OSCP

**Preparation with your existing knowledge:**
- Review: Nmap, Gobuster, Metasploit basics, web app testing, privilege escalation
- Practice: Complete INE free labs + solve 10 easy HTB machines
- Report writing: You'll need to document findings
- Mock exam: TCM Security PNPT labs or HTB Starting Point

### CERT 2 (Alternative): CompTIA Security+
**Get this: Month 6-7 (if eJPT budget is tight)**

| Detail | Info |
|--------|------|
| Cost | ~₹30,000-35,000 (exam voucher) |
| Difficulty | Beginner (theory-heavy, MCQs + performance-based questions) |
| Exam Format | 90 MCQs + PBQs, 90 minutes, passing score 750/900 |
| Validity | 3 years (requires renewal) |
| Prep Time | 6-8 weeks of focused study |
| Industry Value | Globally recognized, but theory-focused. Good for HR filters. |

**When to choose Security+ over eJPT:**
- If you're also considering SOC/blue team roles
- If the company specifically lists Security+ as requirement
- If you want a vendor-neutral globally recognized cert

**Free/Cheap Prep Resources:**
- Professor Messer's Security+ course (FREE on YouTube)
- Jason Dion practice exams (Udemy, ~₹400-600 on sale)
- CompTIA CertMaster Practice (included with some exam bundles)

---

### CERT 3: OSCP (Offensive Security Certified Professional)
**Target: Year 2 (Month 15-18)**

| Detail | Info |
|--------|------|
| Cost | ~₹1,00,000 - 1,50,000 (90-day lab access + exam) |
| Difficulty | Hard (24-hour practical exam, 3 machines + AD set + bonus) |
| Exam Format | Fully practical — pentest 5 targets in 24 hours + report |
| Prep Time | 3-6 months of serious preparation |
| Industry Value | **Gold standard** for pentesting. Opens doors everywhere. |

**Why NOT now:**
- Too expensive without income
- You need more practice to pass reliably
- Failing costs money (exam retake: ~₹50,000+)
- Get a job first → let company sponsor OSCP or save up

**When you're ready for OSCP (checklist):**
- [ ] 40+ HTB machines solved independently
- [ ] Can exploit buffer overflow consistently
- [ ] Strong AD attack chain skills
- [ ] Can write a professional pentest report
- [ ] Have ₹1.5L+ saved for the course
- [ ] Can dedicate 3-4 months of focused prep

---

### CERT 4 (Specialist): CRTP (Certified Red Team Professional)
**Target: After OSCP or as alternative (Year 2)**

| Detail | Info |
|--------|------|
| Cost | ~₹30,000-40,000 (Altered Security) |
| Difficulty | Intermediate (AD-focused, 24-hour practical exam) |
| Exam Format | Practical AD attack chain | 
| Prep Time | 4-6 weeks with your AD background |
| Industry Value | Excellent for AD-focused roles. Indian-made, well-respected. |

**Why consider CRTP:**
- Created by Nikhil Mittal (Indian security researcher)
- Cheaper than OSCP
- Your AD lab experience gives you a head start
- Very practical, no theory dumping

---

## 🟢 Budget-Friendly Certification Path

### If You Have ₹30,000-40,000 (Recommended Minimum)
```
Month 5-6:  eJPT (₹28-35K) → Pass HR filters, prove practical skills
Year 2:     OSCP (₹1-1.5L) → After getting a job, save up or get sponsored
```

### If You Have ₹15,000-25,000
```
Month 6-7:  CompTIA Security+ with self-study (₹30-35K exam, find discount vouchers)
            OR wait 1 more month to save for eJPT
Year 2:     CRTP (₹30-40K) → Cheaper alternative to OSCP
```

### If You Have Less Than ₹15,000
```
Month 1-8:  Focus on FREE proof-of-work (bug bounty, GitHub, blog, CTFs)
Month 9+:   Save from freelance/job income for eJPT
```

> **Hard truth:** Having ZERO certifications is survivable IF your portfolio is exceptional. Having ONE practical cert + good portfolio is the sweet spot.

---

## ❌ Certifications NOT Worth It Right Now

| Certification | Cost | Why to Skip |
|---------------|------|-------------|
| **CEH (Certified Ethical Hacker)** | ₹50,000-1,00,000 | Overpriced, multiple-choice only, outdated content. HR loves it but professionals don't respect it. Your money is better spent on eJPT + OSCP. |
| **CISSP** | ₹60,000+ | Requires 5 years experience. You don't qualify. It's for managers, not pentesters. |
| **CompTIA CySA+** | ₹35,000+ | Blue team focused. If you want pentesting, skip this. |
| **CompTIA PenTest+** | ₹35,000+ | Theory-heavy version of pentesting. eJPT is better for practical skills. |
| **CISM/CISA** | ₹40,000+ | Management/audit certifications. Years away from being relevant. |
| **AWS Security Specialty** | ₹25,000 | Too specialized for entry level. Get basic cloud knowledge instead. |
| **Any ₹5,000-10,000 Udemy 'certification'** | Varies | These are course completions, NOT industry certifications. Don't list them on your resume. |

---

## 💰 Cost Summary Table

| Certification | Cost (INR) | When | ROI Rating | Your Priority |
|---------------|-----------|------|------------|---------------|
| eJPT v2 | ₹28-35K | Month 5-6 | ⭐⭐⭐⭐⭐ | **#1 Priority** |
| CompTIA Security+ | ₹30-35K | Month 6-7 | ⭐⭐⭐⭐ | #2 (alternative to eJPT) |
| CRTP | ₹30-40K | Year 2 | ⭐⭐⭐⭐ | #3 (after first job) |
| OSCP | ₹1-1.5L | Year 2 | ⭐⭐⭐⭐⭐ | #4 (career-defining) |
| CEH | ₹50K-1L | Never | ⭐ | Skip |
| CISSP | ₹60K+ | Year 5+ | N/A | Skip for now |

---

## 🎯 Free Alternatives That Are Almost As Good

These don't replace certifications on a resume, but they prove skills:

| Free Credential | How to Get It | Resume Value |
|----------------|--------------|---------------|
| HackerOne/Bugcrowd profile with findings | Find valid bugs | Very High |
| HTB Pro Hacker rank | Solve 20+ machines on HTB | High |
| TryHackMe Top 1% | Complete learning paths | Medium |
| PortSwigger labs completion | Screenshot your progress | Medium |
| Published security research | Blog post on original research | High |
| Open-source security tool on GitHub | Build something useful | Very High |
| CTF competition rankings | Participate and place | Medium |

> **Pro tip:** A HackerOne profile with 3 valid findings is worth MORE than a CEH certification in the eyes of any hiring manager who actually understands security.

---

*Previous: [05 — Practical Labs Roadmap ←](./05_practical_labs_roadmap.md) | Next: [07 — Resume & Portfolio Strategy →](./07_resume_portfolio_strategy.md)*
