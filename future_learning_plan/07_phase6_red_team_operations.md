# 🕵️ Phase 6: Red Team Operations (Months 21-24)

> **"A pentester finds vulnerabilities. A red teamer simulates the adversary. The difference is methodology, stealth, and operational security."**

---

## 🎯 Objective

Learn to operate like a real-world adversary — evading defenses, maintaining persistence, and achieving mission objectives while remaining undetected.

---

## 📅 Timeline: 4 Months

| Week | Focus | Deliverable |
|---|---|---|
| 1-4 | AV/EDR Evasion Fundamentals | Custom loader that bypasses Defender |
| 5-8 | Command & Control Infrastructure | Operational C2 setup with OPSEC |
| 9-12 | Full Adversary Simulation | Complete red team operation (lab) |
| 13-16 | Reporting & Purple Team | Professional report + detection mapping |

---

## 1️⃣ AV/EDR Evasion

### Understanding Detection

| Detection Type | How It Works | Evasion Approach |
|---|---|---|
| **Signature-based** | Known byte patterns | Obfuscation, encryption, custom code |
| **Heuristic** | Behavioral patterns | API unhooking, indirect syscalls |
| **AMSI** | Script scanning in memory | Patch AMSI, obfuscate scripts |
| **ETW** | Event tracing and logging | Patch ETW, avoid triggering events |
| **Userland Hooks** | DLL hooking of ntdll.dll | Direct syscalls, unhooking |
| **Kernel Callbacks** | Kernel-level process/thread monitoring | Driver-based evasion (advanced) |
| **Memory Scanning** | Scan process memory for IOCs | Encryption, stomping, module overloading |
| **Behavioral Analysis** | AI/ML-based pattern detection | Mimic legitimate behavior |

### Payload Development

| Technique | Description |
|---|---|
| **Custom Shellcode Loader** | Write loader in C/C++/Go/Rust |
| **Process Injection** | Inject shellcode into legitimate processes |
| **DLL Sideloading** | Abuse legitimate app's DLL search order |
| **Module Stomping** | Overwrite legitimate DLL's .text section |
| **Direct Syscalls** | Bypass ntdll hooks by calling syscalls directly |
| **Indirect Syscalls** | Call syscalls through legitimate ntdll code |
| **Sleep Obfuscation** | Encrypt implant memory during sleep |
| **PPID Spoofing** | Fake parent process for legitimacy |
| **Unhooking** | Map fresh ntdll and replace hooked version |

### Exercises

1. Write a basic shellcode loader in C that bypasses Windows Defender
2. Implement process injection (CreateRemoteThread, NtMapViewOfSection)
3. Perform DLL sideloading with a signed Microsoft binary
4. Implement direct syscalls to bypass userland hooks
5. Write a sleep obfuscation routine (encrypt beacon memory during sleep)
6. Test your payload against top EDR solutions in a lab

### Development Languages

| Language | Why |
|---|---|
| **C/C++** | Full Windows API access, most control |
| **Go** | Single binary, cross-compilation, growing C2 ecosystem |
| **Rust** | Memory safety, harder to reverse, growing adoption |
| **Nim** | Python-like syntax, compiles to C, small binaries |
| **C#** | .NET for Windows, inline assembly, reflection |

### Resources

- 📖 **Malware Development for Ethical Hackers** — Alexandre Borges
- 🔗 **MaldevAcademy** — maldevacademy.com (structured malware dev course)
- 🔗 **ired.team** — Red team notes with code examples
- 🎓 **Sektor7 Courses** — RED TEAM Operator series

---

## 2️⃣ Command & Control (C2) Infrastructure

### C2 Frameworks

| Framework | Type | Key Features |
|---|---|---|
| **Cobalt Strike** | Commercial | Industry standard, Beacon, Malleable C2 |
| **Mythic** | Open Source | Multi-agent, modern web UI, extensible |
| **Havoc** | Open Source | Modern C2, demon agent, good evasion |
| **Sliver** | Open Source | Go-based, implant generation, multi-protocol |
| **Brute Ratel** | Commercial | Advanced evasion, designed to bypass EDR |

### C2 Infrastructure OPSEC

```
Operational Architecture:
                                                              
    ┌──────────┐     ┌──────────────┐     ┌───────────────┐   ┌──────────┐
    │  Target   │────►│  Redirector  │────►│  Team Server  │──►│ Operator │
    │  Network  │     │  (CDN/Cloud) │     │  (Backend)    │   │ Workstat │
    └──────────┘     └──────────────┘     └───────────────┘   └──────────┘

Key OPSEC Principles:
├── Never expose team server directly
├── Use redirectors (Cloudflare Workers, AWS CloudFront, Azure CDN)
├── Domain fronting / domain borrowing where possible
├── Use legitimate-looking domains (aged, categorized)
├── Separate infrastructure for different operations
├── Rotate infrastructure regularly
└── Monitor your own infrastructure for detection
```

### Communication Channels

| Channel | Stealth Level | Notes |
|---|---|---|
| **HTTPS** | 🟢 Good | Standard web traffic, blend with browsing |
| **DNS** | 🟢 Good | Slow but hard to block |
| **SMB** | 🟡 Medium | Internal lateral movement, named pipes |
| **WMI** | 🟡 Medium | Windows-native, less monitored |
| **DoH/DoT** | 🟢 Good | Encrypted DNS queries |

### Exercises

1. Set up Mythic or Sliver C2 in a lab
2. Configure redirectors using cloud services
3. Create a Malleable C2 profile that mimics legitimate web traffic
4. Operate through an engagement: initial access → persistence → objective
5. Test detection — run your traffic through a SIEM and tune your profile

---

## 3️⃣ Full Adversary Simulation

### Red Team Kill Chain

```
Phase 1: Reconnaissance
├── OSINT on target organization
├── Identify external attack surface
├── Social engineering targets
└── Technology stack identification

Phase 2: Initial Access
├── Phishing (spear-phishing with pretext)
├── External exploitation
├── Supply chain compromise
└── Physical access

Phase 3: Execution & Persistence
├── Payload execution on target
├── Establish C2 channel
├── Install persistence mechanism
└── Verify callback and stability

Phase 4: Privilege Escalation
├── Local privilege escalation
├── Credential harvesting
├── Token manipulation
└── Exploit misconfiguration

Phase 5: Lateral Movement
├── Move to high-value targets
├── Domain escalation
├── Access sensitive systems
└── Minimize detection footprint

Phase 6: Objective Completion
├── Access crown jewels (data, systems)
├── Demonstrate impact
├── Document attack chain
└── Prove business risk

Phase 7: Reporting & Debrief
├── Executive summary
├── Technical details with evidence
├── Detection gaps mapped to MITRE ATT&CK
└── Remediation recommendations
```

### MITRE ATT&CK Framework

Map every technique you use to MITRE ATT&CK:
- Learn the matrix: Tactics → Techniques → Sub-techniques
- Document your operations using ATT&CK IDs
- Understand detection opportunities for each technique
- This makes your reports immensely more valuable

### Exercises

1. Conduct a full red team operation in a lab environment
2. Map your attack chain to MITRE ATT&CK
3. Write a professional red team report
4. Conduct a purple team exercise — work with "defender" to improve detections
5. Perform adversary emulation: simulate a real APT group's TTPs

---

## 4️⃣ Social Engineering (Offensive)

### Techniques for Red Teams

| Technique | Description |
|---|---|
| **Spear Phishing** | Targeted email with malicious payload/link |
| **Vishing** | Voice phishing — phone-based social engineering |
| **Pretexting** | Create a believable scenario/identity |
| **Watering Hole** | Compromise a site the target visits |
| **USB Drop** | Malicious USB devices |

### Phishing Infrastructure

| Component | Purpose |
|---|---|
| **GoPhish** | Open-source phishing framework |
| **evilginx2** | Adversary-in-the-middle phishing (steal MFA tokens) |
| **Aged domains** | Pre-categorized domains for legitimacy |
| **Email security** | SPF, DKIM, DMARC — set up correctly for delivery |

---

## 📝 Phase 6 Completion Checklist

- [ ] Built a custom shellcode loader that bypasses Windows Defender
- [ ] Implemented process injection (at least 2 techniques)
- [ ] Performed DLL sideloading with a signed binary
- [ ] Implemented direct or indirect syscalls
- [ ] Set up and operated a C2 framework (Mythic/Sliver/Havoc)
- [ ] Configured redirectors and OPSEC-safe infrastructure
- [ ] Conducted a full red team operation in a lab
- [ ] Mapped operations to MITRE ATT&CK
- [ ] Written a professional red team report
- [ ] Set up phishing infrastructure with GoPhish

---

## 🔗 Platforms

| Platform | Focus |
|---|---|
| **HackTheBox Pro Labs** | RastaLabs, Cybernetics |
| **CRTO Lab** | Certified Red Team Operator — Zero-Point Security |
| **Snap Labs** | Red team practice environments |
| **Custom AD Lab** | Build multi-forest environment with EDR |

---

**Next: `08_phase7_ai_offensive_security.md` →**
