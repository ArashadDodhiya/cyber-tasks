# 🧭 Linux Enumeration for Privilege Escalation (PrivEsc)

When you get a low-privileged shell on a Linux machine (lab, CTF, or authorized pentest), the goal is:

> “What can I use to become root?”

Enumeration = collecting information to find misconfigurations.

---

# 🧠 Step-by-Step Linux PrivEsc Enumeration (Simple & Practical)

I’ll structure this like a real-world workflow.

---

# 1️⃣ Basic System Info

Always start here:

```bash
whoami
id
hostname
uname -a
cat /etc/os-release
```

### Why?

* Know your current user
* Know kernel version (kernel exploits?)
* Know OS version

---

# 2️⃣ Check Sudo Permissions

Very important:

```bash
sudo -l
```

If you see something like:

```
User john may run the following commands:
    (ALL) NOPASSWD: /usr/bin/vim
```

🔥 That’s gold.

Then check GTFOBins for that binary.

---

# 3️⃣ SUID Binaries

Classic check:

```bash
find / -perm -4000 2>/dev/null
```

Look for unusual ones like:

* vim
* nano
* bash
* find
* python
* custom binaries

Then check:
👉 [https://gtfobins.github.io](https://gtfobins.github.io)

---

# 4️⃣ Check for Writable Files

### World-writable files:

```bash
find / -writable -type f 2>/dev/null
```

### World-writable directories:

```bash
find / -writable -type d 2>/dev/null
```

Look for:

* Scripts run by root
* Service files
* Cron scripts

---

# 5️⃣ Cron Jobs (Very Common)

Check:

```bash
crontab -l
ls -la /etc/cron*
cat /etc/crontab
```

If root runs something like:

```
* * * * * root backup.sh
```

Then check if:

* `backup.sh` is writable
* It uses relative paths (PATH hijacking)

---

# 6️⃣ PATH Variable

Check:

```bash
echo $PATH
```

Look for:

* `/tmp`
* `/home/user`
* Any writable directory

Also inspect root scripts that use commands without full path.

---

# 7️⃣ Services Running as Root

```bash
ps aux
```

Look for:

* Custom services
* Strange scripts
* Programs running as root

---

# 8️⃣ Capabilities (Modern PrivEsc)

Sometimes binaries don’t use SUID but use Linux capabilities.

Check:

```bash
getcap -r / 2>/dev/null
```

Example:

```
/usr/bin/python3 = cap_setuid+ep
```

That can be abused.

---

# 9️⃣ Check for Passwords

Look in:

```bash
cat /etc/passwd
cat /etc/shadow   # usually denied
```

Search for hardcoded passwords:

```bash
grep -Ri "password" /home 2>/dev/null
```

Check:

* Config files
* Backup files
* .bash_history

---

# 🔟 Kernel Exploits (Last Resort)

Check kernel version:

```bash
uname -r
```

Search for known exploits (only in lab or authorized test).

Modern systems are usually patched.

---

# 🛠️ Automated Enumeration Tools (Ethical Use Only)

In labs/pentests, these help:

* LinPEAS
* LinEnum
* Linux Exploit Suggester
* pspy (for process monitoring)

Always use only with permission.

---

# 🧠 Common PrivEsc Vectors (Exam / CTF Focus)

Most common real-world findings:

✔️ Misconfigured sudo
✔️ Writable cron jobs
✔️ SUID binaries
✔️ PATH hijacking
✔️ Capabilities abuse
✔️ Weak file permissions
✔️ Hardcoded credentials

---

# 🏆 Simple PrivEsc Mindset

Ask these questions:

1. Can I run something as root? → `sudo -l`
2. Is something running as root that I can modify?
3. Is there a file that runs with elevated privileges?
4. Can I trick a script into running my code?

---

# ⚠️ Ethical Reminder

Only perform privilege escalation:

* In CTFs
* On your lab machines
* During authorized penetration tests

Never on production systems without written permission.

---

If you want, I can:

* Give you a structured **PrivEsc checklist**
* Simulate a CTF scenario and solve it step by step
* Explain real-world pentest methodology
* Help you build a Linux PrivEsc lab at home

Tell me your level (beginner/intermediate/advanced) and I’ll tailor it 🔐💻
