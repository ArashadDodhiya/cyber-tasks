# 🎯 How to Crack This Pentester Interview — Master Strategy

> *"They're not testing what you know. They're testing how you think."*

---

## The Two Rounds — What They Actually Want

### Round 1: Lab Round (50 Questions, 80% Pass)

| Detail | Info |
|--------|------|
| **Questions** | 50 lab-based challenges |
| **Pass Score** | 80% (40/50 correct) |
| **Time** | Likely timed |
| **What they test** | Can you actually find and exploit vulns? |
| **Where to prepare** | PortSwigger Academy, DVWA, HackTheBox |

**Strategy for Lab Round:**
1. Speed + accuracy — you need 40/50, so you can afford 10 mistakes max
2. Know your payloads by heart — don't waste time Googling
3. Know tool shortcuts (Burp Suite, browser DevTools)
4. Practice under time pressure — simulate timed lab sessions
5. Focus on the top categories: SQLi, XSS, IDOR, Auth, File Upload, Access Control

---

### Round 2: Manager Round (The Hard One)

This is where they destroy candidates who "got lucky" in the lab round. They want to know:

| What They Ask | What They're Actually Testing |
|---------------|-------------------------------|
| "How did you solve this lab?" | Can you articulate your methodology? |
| "Why did you choose this approach?" | Do you understand the *why*, not just the *what*? |
| "Explain each part of this payload" | Do you actually understand what you're typing? |
| "What if there's a WAF?" | Can you adapt when things don't work? |
| "What would you try next?" | Do you have depth beyond the first trick? |
| "Walk me through your thinking" | Is your thought process systematic or random? |

---

## The Manager Round Framework — How to Answer ANY Question

### The STAR-H Method (Modified for Pentesters)

```
S — Situation:    What was the target/scenario?
T — Thinking:     What was your initial assessment? What clues led you?
A — Action:       What did you actually do? (be specific with payloads/tools)
R — Result:       What happened? Did it work? What was the impact?
H — Hardening:    What if X? What alternative approach? WAF bypass?
```

### Example of a PERFECT Answer:

**Q: "You found a SQL injection in a search field. Walk me through how."**

**Bad Answer:** "I put a single quote and it broke, then I used sqlmap."

**PERFECT Answer:**

> "First, I noticed the search feature was returning results from a backend database, so I hypothesized there might be a SQL query building the response. *(Thinking)*
>
> I started by injecting a single quote (`'`) in the search parameter. The application returned a 500 error instead of a normal 'no results' page. This told me the input is reaching the SQL query without proper sanitization. *(Action — identifying injection point)*
>
> To confirm, I injected `' OR '1'='1` and got all results back, then `' AND '1'='2` which returned zero results. The boolean difference confirmed SQL injection. *(Action — confirming with boolean logic)*
>
> Next, I determined it was MySQL by injecting `' AND @@version-- -` and observing the behavior. Then I used a UNION-based approach: first finding the column count with `ORDER BY` (incremented until error), then `UNION SELECT NULL,NULL,...` to match columns, and finally extracting table names from `information_schema.tables`. *(Action — systematic exploitation)*
>
> If there were a WAF blocking keywords like `UNION` or `SELECT`, I'd try case variation (`uNiOn SeLeCt`), inline comments (`UN/**/ION`), or URL double-encoding. For blind SQLi behind a WAF, I'd use time-based payloads with `SLEEP()` or conditional `BENCHMARK()`. *(Hardening — showing adaptability)*
>
> I chose manual testing first because sqlmap generates a lot of noise and in a real pentest, you want to understand the injection before automating. I'd use sqlmap only after confirming the injection type and crafting a tamper script if needed." *(Why this approach)*

**See the difference?** Every step is explained. Every choice has a reason. They can follow your brain.

---

## The 5 Rules That Will Make You Stand Out

### Rule 1: Never Just Say "I Used a Tool"

❌ "I used sqlmap to dump the database"
✅ "I confirmed the injection type was UNION-based by testing column count with ORDER BY, then I used sqlmap with the `--technique=U --dbms=mysql` flags because I'd already identified those details manually"

### Rule 2: Always Explain the WHY Before the WHAT

❌ "I injected `<script>alert(1)</script>` in the search box"
✅ "The search results reflected my input directly in the HTML response without encoding, so I tested whether the application would render HTML by injecting a benign tag first, then escalated to JavaScript execution"

### Rule 3: Show You Think About Defenses

After every answer, preemptively address:
- "If there was a WAF, I would..."
- "If the output was encoded, I would..."
- "If it was a blind scenario, I would..."

### Rule 4: Show You Think About Impact

Don't stop at "I found XSS." Continue with:
- "This stored XSS fires in the admin panel, meaning I could steal admin sessions"
- "Combined with the lack of HttpOnly flag on cookies, this is account takeover"
- "I could chain this with the CSRF vulnerability I found earlier to..."

### Rule 5: Admit What You Don't Know — Smartly

If you don't know something:
❌ "I don't know"
✅ "I haven't encountered that specific scenario, but my approach would be to first understand the encoding context, then research specific bypass techniques for that encoding. My general methodology would be..."

---

## Daily Practice Schedule (Before the Interview)

### Week 1-2: Build Speed for Lab Round

| Day | Activity | Time |
|-----|----------|------|
| Morning | 3 PortSwigger labs (timed — 15 min each) | 45 min |
| Afternoon | Review + write explanation for each lab | 30 min |
| Evening | Read 2 HackerOne disclosed reports | 20 min |

### Week 3: Build Explanation Muscle for Manager Round

| Day | Activity | Time |
|-----|----------|------|
| Morning | Solve 2 labs, but record yourself explaining aloud | 40 min |
| Afternoon | Practice answering scenario questions (use the files below) | 30 min |
| Evening | Review your explanations — are they clear? Would a non-tech person follow? | 20 min |

### Week 4: Mock Interview

| Day | Activity | Time |
|-----|----------|------|
| Daily | Solve 1 lab + explain it as if in an interview | 30 min |
| Daily | Random scenario question — answer with STAR-H method | 20 min |
| Daily | Review cheat sheets for instant recall | 10 min |

---

## Folder Structure — Your Interview Arsenal

```
mindset/interview_cracking_strategy/
├── 00_how_to_crack_this_interview.md     ← YOU ARE HERE
├── 01_sql_injection_scenarios.md         ← SQLi interview scenarios + answers
├── 02_xss_scenarios.md                   ← XSS interview scenarios + answers
├── 03_idor_scenarios.md                  ← IDOR interview scenarios + answers
├── 04_auth_session_scenarios.md          ← Auth/Session flaw scenarios
├── 05_file_upload_scenarios.md           ← File upload bypass scenarios
├── 06_access_control_scenarios.md        ← Access control scenarios
├── 07_payload_explanation_drills.md      ← "Explain this payload" practice
├── 08_waf_bypass_encyclopedia.md         ← WAF bypass techniques
├── 09_thinking_process_templates.md      ← Templates for explaining your thinking
└── 10_rapid_fire_cheatsheet.md           ← Quick recall during lab round
```

Each file contains:
- 🎯 **Scenario questions** (what the interviewer asks)
- ✅ **Model answers** (how to answer perfectly)
- 🔥 **Follow-up challenges** (what they'll ask next)
- 💡 **Key insights** (what separates good from great answers)
- ⚠️ **Common mistakes** (what NOT to say)

---

## Your Mindset Going In

```
┌─────────────────────────────────────────────────┐
│                                                 │
│   "I don't just know what to do.                │
│    I know WHY I do it,                          │
│    I know WHAT ELSE I could do,                 │
│    and I know WHEN each approach is best."       │
│                                                 │
└─────────────────────────────────────────────────┘
```

**That's what they want to hear. That's what these files will teach you. Let's go.**

---

> **Next: Open `01_sql_injection_scenarios.md` and start drilling.**
