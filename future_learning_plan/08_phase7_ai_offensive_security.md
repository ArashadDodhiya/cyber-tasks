# 🤖 Phase 7: AI Offensive Security (Months 25-27)

> **"The future offensive security professional doesn't just hack systems — they hack AI AND use AI to hack better."**

---

## 🎯 Objective

Master two domains: (1) Attacking AI systems themselves, and (2) Using AI to enhance your offensive capabilities.

---

## 📅 Timeline: 3 Months

| Week | Focus | Deliverable |
|---|---|---|
| 1-2 | AI/ML Fundamentals for Hackers | Understand LLM architecture enough to attack it |
| 3-4 | Prompt Injection Mastery | Bypass 5+ prompt injection defenses |
| 5-6 | AI Agent & RAG Attacks | Exploit an AI agent's tool-calling abilities |
| 7-8 | AI-Powered Recon & Exploitation | Build AI-assisted recon tool |
| 9-10 | Building Offensive AI Tools | AI payload mutator or AI-assisted exploit analyzer |
| 11-12 | AI Red Teaming & Research | Write up an AI attack methodology |

---

## 1️⃣ AI Fundamentals (For Attackers)

### What You MUST Understand

| Concept | Why It Matters for Offense |
|---|---|
| **Tokens & Tokenization** | Understand how input is processed — basis for injection |
| **Context Windows** | Know input limits and how to manipulate context |
| **Embeddings** | How semantic search works → poisoning vectors |
| **Attention Mechanism** | How models "focus" → manipulate attention |
| **System Prompts** | The "instructions" you're trying to override |
| **Temperature/Top-p** | How randomness affects exploitability |
| **Fine-tuning** | How custom models are made → attack training data |
| **RAG (Retrieval Augmented Generation)** | External knowledge → injection point |
| **Tool Calling / Function Calling** | AI executing code → massive attack surface |
| **AI Agents** | Autonomous AI with tools → autonomous exploitation |

### Resources

- 📖 **Build a Large Language Model (From Scratch)** — Sebastian Raschka
- 🎓 **Andrej Karpathy: Neural Networks Zero to Hero** (YouTube)
- 🔗 **OWASP Top 10 for LLM Applications 2025**
- 📄 **Anthropic, OpenAI, Google safety research papers**

---

## 2️⃣ Attacking AI Systems

### Prompt Injection

| Type | Description |
|---|---|
| **Direct Prompt Injection** | Attacker directly crafts malicious input to override system prompt |
| **Indirect Prompt Injection** | Malicious content in data the AI retrieves (web, docs, emails) |
| **Jailbreaking** | Bypass safety guardrails to get harmful outputs |
| **Prompt Leaking** | Extract the system prompt / instructions |
| **Context Manipulation** | Gradually shift AI behavior through conversation |

### Advanced AI Attacks

| Attack | Description |
|---|---|
| **RAG Poisoning** | Inject malicious content into knowledge bases the AI retrieves from |
| **Agent Hijacking** | Manipulate AI agent to call tools maliciously (file access, API calls, code execution) |
| **Tool Abuse** | Exploit tool-calling to perform unauthorized actions |
| **Memory Poisoning** | Corrupt persistent memory in multi-turn AI systems |
| **Data Exfiltration** | Trick AI into revealing training data or user data |
| **Model Extraction** | Steal model weights or architecture through API queries |
| **Adversarial Examples** | Craft inputs that cause AI to misclassify (images, text) |
| **Training Data Poisoning** | Insert backdoors into training data for future exploitation |
| **Supply Chain** | Compromise model weights, plugins, or extensions |

### Exercises

1. Extract system prompts from 5+ AI chatbots using prompt injection
2. Jailbreak a safety-filtered model using DAN, multi-persona, or encoding techniques
3. Poison a RAG system — inject malicious documents that change AI responses
4. Hijack an AI agent's tool-calling to access unauthorized files
5. Exfiltrate data from an AI system through markdown image injection
6. Complete Gandalf (Lakera) prompt injection CTF — all levels

### Resources

- 🧪 **Gandalf (Lakera)** — gandalf.lakera.ai (prompt injection CTF)
- 🧪 **Prompt Airlines** — promptairlines.com (realistic AI vuln lab)
- 🧪 **Damn Vulnerable LLM Agent** — Practice AI agent attacks
- 🔗 **OWASP LLM Top 10** — owasp.org
- 📄 **"Not what you've signed up for"** — Greshake et al. (indirect prompt injection paper)

---

## 3️⃣ Using AI for Offense

### AI-Enhanced Reconnaissance

| Use Case | How AI Helps |
|---|---|
| **Subdomain Discovery** | AI analyzes patterns and predicts subdomains |
| **Technology Fingerprinting** | LLM analyzes response headers and content |
| **Vulnerability Correlation** | AI matches discovered services to known CVEs |
| **OSINT Analysis** | Process and correlate massive OSINT data |
| **Report Generation** | Auto-generate preliminary pentest reports |
| **Code Review** | AI-assisted source code vulnerability analysis |

### AI-Enhanced Exploitation

| Use Case | How AI Helps |
|---|---|
| **Payload Mutation** | AI generates variations to bypass WAF/filters |
| **Exploit Analysis** | Feed CVE details → AI explains exploitation path |
| **Reverse Engineering** | AI explains decompiled code |
| **Phishing** | AI generates convincing pretexts and templates |
| **Fuzzer Guidance** | AI predicts which inputs are more likely to crash |

### Build These AI Offensive Tools

| Tool | What It Does |
|---|---|
| **AI Recon Assistant** | Feed target → get organized recon with AI analysis |
| **AI Payload Mutator** | Input a blocked payload → get 10 bypass variations |
| **AI Exploit Analyzer** | Feed a CVE → get exploitation steps and PoC guidance |
| **AI Code Reviewer** | Feed source code → get vulnerability report |
| **AI Report Writer** | Feed findings → get formatted pentest report |
| **AI Log Analyzer** | Feed access logs → identify attack patterns |

### Tech Stack for AI Offensive Tools

```
LLM APIs: OpenAI, Anthropic Claude, local models (Ollama + Llama)
Frameworks: LangChain, LlamaIndex (for RAG), CrewAI (for agents)
Vector DBs: ChromaDB, Pinecone (for knowledge bases)
Local Models: Ollama + Mistral/Llama for offline operation
Python: Primary language for AI tool development
```

### Exercises

1. Build an AI-powered recon tool that processes Nmap output and suggests next steps
2. Create a payload mutation tool that generates WAF bypass variations using an LLM
3. Build a RAG system with pentest methodology docs — query it during engagements
4. Create an AI agent that can perform basic web reconnaissance autonomously
5. Build a vulnerability analysis tool: input CVE number → get exploitation guide

---

## 4️⃣ AI Red Teaming

### What Is AI Red Teaming?

Testing AI systems for safety, security, and alignment failures. Companies like OpenAI, Google, Anthropic, and Microsoft all hire AI red teamers.

### AI Red Teaming Methodology

```
1. Scope Definition
   ├── What AI system are you testing?
   ├── What are the safety boundaries?
   ├── What constitutes a "break"?
   └── What tools/APIs do you have access to?

2. Threat Modeling
   ├── Who are the adversaries?
   ├── What are their goals?
   ├── What access do they have?
   └── What would be high-impact failures?

3. Attack Execution
   ├── Prompt injection (direct + indirect)
   ├── Jailbreaking
   ├── Data extraction
   ├── Tool/function abuse
   ├── Multi-turn manipulation
   └── Adversarial examples

4. Impact Assessment
   ├── Severity of the bypass
   ├── Reproducibility
   ├── User impact
   └── Remediation difficulty

5. Reporting
   ├── Clear reproduction steps
   ├── Impact analysis
   ├── Remediation recommendations
   └── Detection capabilities
```

---

## 📝 Phase 7 Completion Checklist

- [ ] Understand LLM architecture at a technical level
- [ ] Extracted system prompts from 5+ AI chatbots
- [ ] Completed Gandalf prompt injection CTF (all levels)
- [ ] Poisoned a RAG system and changed AI output
- [ ] Hijacked an AI agent's tool-calling capabilities
- [ ] Built an AI-powered recon tool
- [ ] Built an AI payload mutation tool
- [ ] Built a RAG-based pentest assistant
- [ ] Performed AI red teaming on at least one AI system
- [ ] Written up an AI attack methodology document

---

**Next: `09_phase8_bug_bounty_real_world.md` →**
