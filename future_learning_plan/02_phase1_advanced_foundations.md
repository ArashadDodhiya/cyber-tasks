# 🧱 Phase 1: Advanced Foundations (Months 1-3)

> **"You can't hack what you don't understand. The deeper your systems knowledge, the more creative your attacks."**

---

## 🎯 Objective

Go beyond tool usage. Understand HOW systems work at a deep level so you can find novel ways to break them.

---

## 📅 Timeline: 3 Months

| Week | Focus Area | Deliverable |
|---|---|---|
| 1-2 | TCP/IP Deep Dive | Wireshark analysis of 10+ protocols |
| 3-4 | Linux Internals | Custom kernel module + syscall tracing |
| 5-6 | Windows Internals | Process injection PoC + WinAPI scripting |
| 7-8 | Python Offensive Scripting | 3 custom offensive tools |
| 9-10 | Bash + Go for Offense | Network scanner in Go |
| 11-12 | Assembly Fundamentals | Read and write basic x86-64 assembly |

---

## 1️⃣ Networking Deep Dive

### What to Master

| Topic | Depth Level |
|---|---|
| **TCP/IP Stack** — every header field, RST attacks, sequence prediction | Expert |
| **DNS** — zone transfers, tunneling, rebinding, cache poisoning | Expert |
| **HTTP/1.1 & HTTP/2** — request smuggling, header injection, h2c smuggling | Expert |
| **TLS/SSL** — pinning bypasses, downgrade attacks, mTLS | Advanced |
| **ARP/Layer 2** — ARP spoofing, VLAN hopping, STP attacks | Advanced |
| **IPv6** — RA flooding, IPv6 MITM, dual-stack attacks | Intermediate |

### Hands-On Exercises

1. Capture a full TCP 3-way handshake in Wireshark — identify every flag and sequence number
2. Capture and analyze a TLS handshake — identify cipher negotiation
3. Set up DNS tunneling with iodine or dnscat2
4. Perform ARP spoofing with ettercap and capture credentials from HTTP traffic
5. Craft custom packets with Scapy — SYN scan, DNS queries, ICMP tunneling

### Tools to Master

| Tool | Purpose |
|---|---|
| **Wireshark** | Deep packet analysis |
| **tcpdump** | CLI packet capture |
| **Scapy** | Packet crafting (Python) |
| **Responder** | LLMNR/NBT-NS poisoning |
| **Bettercap** | Network attacks framework |

### Resources

- 📖 **TCP/IP Illustrated Vol 1** — W. Richard Stevens
- 🧪 **malware-traffic-analysis.net** — Wireshark challenges
- 🎯 **TryHackMe: Wireshark** rooms

---

## 2️⃣ Linux Internals

### What to Master

| Topic | Details |
|---|---|
| **Filesystem** | Everything is a file, /proc, /sys, /dev, inode structure |
| **Process Model** | fork(), exec(), PID namespaces, process tree |
| **Memory** | Virtual memory, mmap, /proc/[pid]/maps, stack vs heap |
| **Permissions** | DAC, SUID/SGID deep understanding, capabilities, ACLs |
| **Kernel** | Syscalls, kernel modules, eBPF basics |
| **Namespaces & cgroups** | Container isolation foundations |
| **SELinux/AppArmor** | MAC — how to identify and bypass |

### Key Commands to MASTER

```bash
strace -p <pid>                    # Trace syscalls
ltrace -p <pid>                    # Trace library calls
cat /proc/<pid>/maps               # Memory map
cat /proc/<pid>/environ            # Environment variables
getcap -r / 2>/dev/null           # Find binaries with capabilities
find / -perm -4000 2>/dev/null    # SUID binaries
ss -tlnp                          # All listening ports
```

### Exercises

1. Explore /proc/self — understand every file
2. Trace syscalls of a running process with strace and ltrace
3. Create a minimal container using unshare (no Docker)
4. Attempt a container escape from a privileged container
5. Understand Docker socket exposure → host compromise

### Resources

- 📖 **How Linux Works** — Brian Ward
- 📖 **The Linux Programming Interface** — Michael Kerrisk
- 🎓 **pwn.college** — Systems security course
- 🎯 **OverTheWire: Bandit → Narnia → Leviathan**

---

## 3️⃣ Windows Internals

### What to Master

| Topic | Details |
|---|---|
| **Architecture** | User mode vs Kernel mode, HAL, Executive |
| **Processes & Threads** | PEB, TEB, process creation, DLL loading |
| **Registry** | Hive structure, persistence locations, SAM database |
| **Tokens & Privileges** | Access tokens, impersonation, SeDebugPrivilege |
| **Services** | SCM, service exploitation |
| **WMI/COM** | Enumeration, lateral movement, COM hijacking |
| **Named Pipes** | IPC, Potato family attacks |
| **AMSI/ETW** | Understanding and bypassing |

### Exercises

1. Use Process Explorer to examine access tokens
2. Enumerate auto-run locations in registry
3. Extract password hashes from SAM/SYSTEM hives (offline)
4. Analyze DLL load order → identify hijacking opportunities
5. Use Process Monitor to trace file/registry/network activity

### Resources

- 📖 **Windows Internals Part 1 & 2** — Yosifovich, Russinovich
- 📖 **Windows Security Internals** — James Forshaw
- 🎯 **TryHackMe: Windows Internals** rooms

---

## 4️⃣ Programming for Offense

### Python — Your Primary Weapon

Build these projects:

| Project | Skills Practiced |
|---|---|
| **Port Scanner** | Socket programming, threading |
| **Directory Buster** | HTTP requests, concurrency |
| **Subdomain Enumerator** | DNS resolution, API integration |
| **Reverse Shell Handler** | Socket programming, process management |
| **Web Fuzzer** | HTTP, parameter injection |
| **Burp Extension** | Jython, Burp API |

Key libraries: `socket`, `requests`, `scapy`, `paramiko`, `impacket`, `struct`, `asyncio`, `argparse`

### Bash — Your Daily Driver

Master one-liner patterns for recon, enumeration, and quick exploitation.

### Go — For Performance Tools

- Compiles to single static binary (no dependencies on target)
- Cross-compilation is trivial
- Build: concurrent scanner, reverse shell, basic C2 beacon

### C/C++ — For Low-Level Work

- Windows API: CreateProcess, VirtualAlloc, WriteProcessMemory
- Linux syscalls in C
- Foundation for exploit development and shellcode

---

## 📝 Phase 1 Completion Checklist

- [ ] Can explain every field in a TCP header and its security implications
- [ ] Can craft custom packets with Scapy
- [ ] Can trace syscalls and understand process behavior on Linux
- [ ] Can enumerate Windows processes, tokens, and privileges
- [ ] Built 3+ custom offensive tools in Python
- [ ] Can read basic x86-64 assembly
- [ ] Can set up and escape Docker containers
- [ ] Can perform network MITM attacks
- [ ] Completed OverTheWire: Bandit + 2 other wargames

---

## 🔗 Platforms for This Phase

| Platform | What to Do |
|---|---|
| **OverTheWire** | Bandit → Narnia → Behemoth |
| **pwn.college** | Assembly, memory, syscall modules |
| **TryHackMe** | Linux/Windows internals rooms |
| **HackTheBox** | Easy Linux + Easy Windows machines |
| **Exploit Education** | Phoenix for memory corruption basics |

---

**Next: `03_phase2_web_app_mastery.md` →**
