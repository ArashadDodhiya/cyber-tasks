# 🧪 Practical Labs Roadmap

> Labs are where skills become real. This document gives you the EXACT platforms, paths, and order to follow. No more random lab hopping.

---

## 🎯 Platform Strategy: Which Platform, When, Why

| Platform | Cost | Best For | When to Use | Priority |
|----------|------|----------|-------------|----------|
| **PortSwigger Web Security Academy** | Free | Web app vulnerabilities deep dive | Month 1-3 (primary), Month 5-10 (revisit expert labs) | 🔴 Critical |
| **TryHackMe** | Free tier + ₹800/month for premium | Guided learning, theory + practice | Month 1-2 (fill gaps), then occasional | 🟡 Important |
| **Hack The Box** | Free tier + ₹800/month for VIP | Realistic machines, interview prep | Month 3-6 (primary), Month 7-12 (ongoing) | 🔴 Critical |
| **VulnHub** | Free | Downloadable VMs, offline practice | Month 3-5 (supplement HTB) | 🟢 Useful |
| **OWASP Juice Shop** | Free | Modern web app exploitation | Month 1-2 | 🟡 Important |
| **DVWA** | Free | Web vuln basics at different security levels | Already done. Revisit with new knowledge only. | 🟢 Done |
| **PentesterLab** | ~₹2500/year | Real-world web vuln exercises | If budget allows, Month 4+ | 🟢 Useful |
| **CTFtime.org** | Free | Weekly CTF competitions | Month 4+ (weekend activity) | 🟡 Important |
| **flaws.cloud** | Free | AWS security | Month 7 | 🟡 Important |
| **CloudGoat** | Free | AWS attack scenarios | Month 7-8 | 🟡 Important |
| **Proving Grounds (OffSec)** | ~₹1500/month | OSCP-style machines | Month 11+ or Year 2 (OSCP prep) | 🟢 Later |

---

## 🌐 PortSwigger Web Security Academy — Complete Path

### Phase 1: Apprentice Labs (Month 1) — Complete ALL
These are your foundation. No skipping.

| Topic | # Labs | Priority | Estimated Time |
|-------|--------|----------|----------------|
| SQL Injection | 16 | 🔴 Critical | 3-4 days |
| Cross-Site Scripting (XSS) | 15 | 🔴 Critical | 3-4 days |
| Cross-Site Request Forgery (CSRF) | 5 | 🟡 Important | 1 day |
| Authentication | 10 | 🔴 Critical | 2-3 days |
| Access Control | 13 | 🔴 Critical | 2-3 days |
| Server-Side Request Forgery (SSRF) | 5 | 🔴 Critical | 1-2 days |
| File Upload Vulnerabilities | 4 | 🟡 Important | 1 day |
| Path Traversal | 6 | 🟡 Important | 1 day |
| OS Command Injection | 5 | 🟡 Important | 1 day |
| Information Disclosure | 5 | 🟡 Important | 1 day |
| Business Logic | 7 | 🟡 Important | 1-2 days |

### Phase 2: Practitioner Labs (Month 2-3) — Complete 80%+
This is where real skill develops.

| Topic | Priority | Notes |
|-------|----------|-------|
| Advanced SQL Injection | 🔴 Critical | Blind SQLi, second-order, filter bypass |
| DOM-based XSS | 🔴 Critical | Most missed by beginners |
| OAuth Authentication | 🟡 Important | Common in real-world apps |
| JWT Attacks | 🔴 Critical | Every modern API uses JWT |
| SSRF Advanced | 🟡 Important | Blind SSRF, filter bypass |
| Server-Side Template Injection | 🟡 Important | Growing attack vector |
| Insecure Deserialization | 🟡 Important | Java background helps here |
| Web Cache Poisoning | 🟢 Useful | Advanced technique |
| HTTP Host Header Attacks | 🟢 Useful | |
| HTTP Request Smuggling | 🟢 Useful | Expert level, can wait |

### Phase 3: Expert Labs (Month 9-12) — Attempt 50%+
These are genuinely hard. Don't feel bad if you can't solve them without hints.

| Topic | Notes |
|-------|-------|
| Prototype Pollution | Emerging attack, very relevant |
| Race Conditions | New labs, very practical |
| Advanced Request Smuggling | Hard but impressive on resume |
| GraphQL Vulnerabilities | Growing attack surface |
| Web LLM Attacks | Newest category, cutting-edge |

**Tracking Target:**
- Month 1: 40+ labs
- Month 3: 80+ labs
- Month 6: 120+ labs
- Month 12: 150+ labs (all apprentice + 80% practitioner + some expert)

---

## 🖥️ Hack The Box — Machine Progression

### Tier 1: Starting Point (Month 3, Week 1)
Complete ALL Starting Point machines (both tiers). These teach the HTB methodology.

### Tier 2: Easy Machines (Month 3-4) — Target: 15 machines
**Linux Easy (do these first):**
- Lame, Jerry, Shocker, Bashed, Nibbles, Beep, Blocky, Mirai, Valentine, Sense

**Windows Easy:**
- Blue, Legacy, Devel, Optimum, Granny, Grandpa, Arctic

### Tier 3: Medium Machines (Month 4-6) — Target: 15 machines
**Linux Medium:**
- Cronos, Nineveh, Solidstate, Poison, Sunday, Tartarsauce

**Windows Medium:**
- Bounty, Jerry (harder path), SecNotes, Querier, Arkham

**Active Directory:**
- Forest, Sauna, Monteverde, Resolute, Cascade

### Tier 4: Hard Machines (Month 9+) — Target: 5-10 machines
Only attempt these after Month 8. They require chaining multiple vulnerabilities.

**AD Hard:**
- Blackfield, Reel, Mantis

**Tracking Target:**
- Month 3: 10 machines
- Month 6: 25 machines
- Month 9: 35 machines
- Month 12: 40+ machines

### HTB Pro Labs (If Budget Allows)
| Lab | Cost | Best For | When |
|-----|------|----------|------|
| Dante | ~₹7,000 | Entry-level network pentesting | Month 6-7 |
| Offshore | ~₹7,000 | AD attack chains | Month 8-9 |
| RastaLabs | ~₹7,000 | Red team ops | Year 2 |

---

## 🔵 TryHackMe — Targeted Paths

Don't do everything on THM. Use it strategically to fill specific gaps.

### Must-Complete Paths
| Path | When | Time | Why |
|------|------|------|-----|
| Jr Penetration Tester | Month 1 (if not done) | 40-50h | Structured pentesting methodology |
| Web Fundamentals | Month 1-2 | 30h | Fill web security gaps |
| Offensive Pentesting | Month 3-4 | 50h | Buffer overflow, priv esc practice |
| Red Team Fundamentals | Month 6-7 | 30h | Understanding red team operations |

### Skip These (For Now)
- SOC Level 1 (only if pivoting to blue team)
- Cyber Defense (blue team focus)
- CompTIA Pentest+ path (certification-focused, not practical enough)

---

## 🏁 CTF Recommendations

### Getting Started with CTFs (Month 4+)

**Weekly Practice:**
- PicoCTF (https://picoctf.org) — beginner-friendly, always available
- OverTheWire Bandit/Natas — Linux + web fundamentals

**Monthly Competitions (CTFtime.org):**
- Look for Jeopardy-style CTFs rated Easy-Medium
- Focus categories: **Web, Crypto, Forensics**
- Recommended: PicoCTF, NahamCon CTF, DiceCTF, CSAW CTF

**Indian CTFs:**
- InCTF (by Amrita University)
- Pragyan CTF
- nullcon HackIM

### CTF Strategy
1. Attempt every web challenge first (your strength)
2. Try crypto/forensics next
3. Skip pwn/reverse until Year 2
4. ALWAYS write up your solutions, even for easy challenges
5. Join a CTF team on Discord for collaboration

---

## 🏠 Vulnerable Applications for Local Practice

| Application | What to Practice | Setup Difficulty |
|-------------|-----------------|------------------|
| DVWA | Web vulns at multiple levels | Easy (Docker) |
| OWASP Juice Shop | Modern web app exploitation | Easy (Docker/Node) |
| WebGoat | Java-based web vulnerabilities | Easy (Docker) |
| Damn Vulnerable Web Services (DVWS) | API security | Medium |
| OWASP crAPI | API security | Medium |
| Vulnerable AD (Orange Cyberdefense) | AD attack practice | Hard (needs VMs) |
| Metasploitable 2/3 | Network exploitation | Easy (VM) |
| Kubernetes Goat | Container security | Medium |
| CloudGoat | AWS attacks | Medium |

### Recommended Lab Setup (You Already Have Most of This)
```
Host Machine
├─ VirtualBox / VMware
│  ├─ Kali Linux (attacker)
│  ├─ Metasploitable 2 (target)
│  ├─ DVWA (Docker on Kali)
│  ├─ Juice Shop (Docker)
│  ├─ Windows Server 2019 DC (AD lab)
│  ├─ Windows 10 Workstation x2 (AD lab)
│  └─ Ubuntu Server (web app testing)
└─ Docker Desktop
   ├─ DVWS
   ├─ crAPI
   └─ WebGoat
```

---

## 📈 Lab Progress Tracker

| Platform | Target (Month 3) | Target (Month 6) | Target (Month 9) | Target (Month 12) |
|----------|------------------|-------------------|-------------------|--------------------|
| PortSwigger | 80 labs | 120 labs | 140 labs | 150+ labs |
| HTB Machines | 15 | 25 | 35 | 40+ |
| TryHackMe Rooms | 40 rooms | 60 rooms | 70 rooms | 80+ rooms |
| VulnHub VMs | 3 | 5 | 7 | 10 |
| CTF Competitions | 0 | 2 | 5 | 8+ |
| Bug Bounty Submissions | 1-2 (VDP) | 5+ | 10+ | 15+ |

---

## 💡 Lab Rules

1. **Never use walkthroughs on first attempt.** Struggle for at least 2 hours before looking at hints.
2. **ALWAYS write notes while solving.** Your future self will thank you.
3. **If you use a walkthrough, redo the machine without it within 1 week.**
4. **Publish writeups for retired HTB machines.** (Don't publish active machine solutions.)
5. **Time yourself.** Track how long machines take. Speed improves with practice.
6. **Focus on methodology, not just flags.** Interviewers ask "how did you approach it?" not "did you get root?"

---

*Previous: [04 — Learning Order & Priorities ←](./04_learning_order_priorities.md) | Next: [06 — Certification Guidance →](./06_certification_guidance.md)*
