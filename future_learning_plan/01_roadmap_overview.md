# 🗺️ Offensive Security Mastery — Roadmap Overview

---

## 📊 Skill Progression Model

```
                    ┌─────────────────────────────────┐
                    │          ELITE TIER              │
                    │   0-Day Researcher / Red Team    │
                    │   Lead / Exploit Developer /     │
                    │   AI Offensive Architect         │
                    └────────────┬────────────────────┘
                                 │
                    ┌────────────┴────────────────────┐
                    │        EXPERT TIER               │
                    │   Red Teamer / Cloud Attacker /  │
                    │   Bug Bounty Pro / Malware Dev   │
                    └────────────┬────────────────────┘
                                 │
                    ┌────────────┴────────────────────┐
                    │      SPECIALIST TIER             │
                    │   Web Expert / AD Expert /       │
                    │   Network Pentester              │
                    └────────────┬────────────────────┘
                                 │
                    ┌────────────┴────────────────────┐
                    │      FOUNDATION TIER ← YOU      │
                    │   Fundamentals Complete /        │
                    │   Tool Usage / Basic Methodology │
                    └─────────────────────────────────┘
```

---

## 🛤️ The Journey — Phase by Phase

```
FOUNDATION ──► SPECIALIST ──► EXPERT ──► ELITE
    │               │            │          │
    ▼               ▼            ▼          ▼
 Deep Systems    Advanced     Red Team    0-Day
 Knowledge       Web/Net/     + Cloud +   Research
 + Scripting     AD Mastery   Evasion     + AI Offense
```

---

## 📅 Master Timeline (28+ Months)

### Year 1: Foundation → Specialist

| Month | Phase | Key Milestone |
|---|---|---|
| 1 | Advanced Foundations | Master TCP/IP at packet level |
| 2 | Advanced Foundations | Linux internals deep dive |
| 3 | Advanced Foundations | Windows internals + Python mastery |
| 4 | Web App Mastery | PortSwigger all labs (Apprentice + Practitioner) |
| 5 | Web App Mastery | API hacking + OAuth/JWT attacks |
| 6 | Web App Mastery | Advanced: HTTP smuggling, cache poisoning |
| 7 | Web App Mastery | Browser internals + client-side attacks |
| 8 | Network & Infra | Advanced network attacks + pivoting mastery |
| 9 | Network & Infra | Active Directory: Kerberos, delegation, trusts |
| 10 | Network & Infra | AD forest attacks + lateral movement chains |
| 11 | Network & Infra | OSCP-level full chain pentests |
| 12 | Cloud Offensive | AWS IAM attacks + metadata exploitation |

### Year 2: Specialist → Expert

| Month | Phase | Key Milestone |
|---|---|---|
| 13 | Cloud Offensive | Kubernetes + container escapes |
| 14 | Cloud Offensive | CI/CD attacks + multi-cloud pivoting |
| 15 | Exploit Dev & RE | Assembly + stack buffer overflows |
| 16 | Exploit Dev & RE | Heap exploitation + format strings |
| 17 | Exploit Dev & RE | ROP chains + shellcode development |
| 18 | Exploit Dev & RE | Windows exploitation + kernel basics |
| 19 | Exploit Dev & RE | Linux kernel exploitation |
| 20 | Exploit Dev & RE | Fuzzing + vulnerability research |
| 21 | Red Team Ops | AV/EDR evasion + custom loaders |
| 22 | Red Team Ops | C2 infrastructure + OPSEC |
| 23 | Red Team Ops | Full adversary simulation operations |
| 24 | Red Team Ops | Purple team exercises + reporting |

### Year 2.5+: Expert → Elite

| Month | Phase | Key Milestone |
|---|---|---|
| 25 | AI Offensive | LLM attacks + prompt injection mastery |
| 26 | AI Offensive | AI agent exploitation + autonomous tools |
| 27 | AI Offensive | Building AI-powered offensive systems |
| 28+ | Bug Bounty | First valid bug submission |
| 29+ | Bug Bounty | Consistent findings, specialization |
| 30+ | Bug Bounty | Writing research, CVEs, conference talks |

---

## 🎯 Key Milestones & Checkpoints

### Checkpoint 1: "Can I Break Web Apps Deeply?" (Month 7)
- [ ] Completed all PortSwigger Practitioner-level labs
- [ ] Found and exploited business logic flaws
- [ ] Exploited OAuth/SAML/JWT in realistic scenarios
- [ ] Understood HTTP/2 attacks and request smuggling
- [ ] Built a custom Burp extension

### Checkpoint 2: "Can I Own a Network?" (Month 11)
- [ ] Compromised a multi-subnet network end-to-end
- [ ] Achieved Domain Admin through multiple AD attack paths
- [ ] Performed Kerberoasting, AS-REP roasting, delegation attacks
- [ ] Pivoted through 3+ network segments
- [ ] Completed OSCP-equivalent challenge

### Checkpoint 3: "Can I Attack the Cloud?" (Month 14)
- [ ] Escalated privileges in AWS/Azure from low-priv user
- [ ] Escaped a Docker container
- [ ] Compromised a Kubernetes cluster
- [ ] Exploited a CI/CD pipeline
- [ ] Accessed secrets from cloud metadata services

### Checkpoint 4: "Can I Write Exploits?" (Month 20)
- [ ] Developed a working buffer overflow exploit
- [ ] Built ROP chains for DEP bypass
- [ ] Written custom shellcode
- [ ] Fuzzed a real application and found a crash
- [ ] Analyzed a real CVE and reproduced the exploit

### Checkpoint 5: "Can I Simulate a Real Adversary?" (Month 24)
- [ ] Bypassed Windows Defender / EDR in a lab
- [ ] Operated a C2 framework with OPSEC
- [ ] Completed a full red team engagement (lab)
- [ ] Created custom implants/loaders
- [ ] Produced a professional red team report

### Checkpoint 6: "Can I Hack AI and Use AI to Hack?" (Month 27)
- [ ] Successfully performed prompt injection on protected LLMs
- [ ] Exploited an AI agent's tool-calling capabilities
- [ ] Built an AI-powered recon/exploitation tool
- [ ] Understood RAG poisoning and data exfiltration from AI systems

### Checkpoint 7: "Can I Find Real Bugs?" (Month 30+)
- [ ] Submitted first valid bug to a VDP
- [ ] Received first bounty payment
- [ ] Published a technical writeup
- [ ] Discovered a novel attack technique or chain

---

## 🔀 Parallel Tracks (Run Alongside Any Phase)

These should be continuous throughout your journey:

| Track | Activities |
|---|---|
| **CTFs** | Weekly CTF participation (HTB, THM, CTFtime) |
| **CVE Study** | Reproduce 1 CVE per week |
| **Tool Building** | Build/improve 1 offensive tool per month |
| **Writing** | 1 technical blog post per month |
| **Community** | Active on Twitter/X infosec, Discord communities |
| **Reading** | Daily: blog posts, advisories, research papers |

---

## 🧠 Learning Methodology

### The 70-20-10 Rule for Offensive Security

| % | Activity | Examples |
|---|---|---|
| **70%** | Hands-on practice | Labs, CTFs, bug bounty, tool building |
| **20%** | Learning from others | Writeups, courses, mentors, conferences |
| **10%** | Formal study | Books, certifications, theory |

### The Exploit-First Approach

For every new topic:
1. **Find a vulnerable target** → set it up or find a lab
2. **Try to exploit it** → attempt before reading
3. **Fail and research** → understand why it didn't work
4. **Exploit successfully** → document the exact steps
5. **Automate it** → write a script or tool
6. **Teach it** → write it up or explain to someone

---

**Next: `02_phase1_advanced_foundations.md` →**
