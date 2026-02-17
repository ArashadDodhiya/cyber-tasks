# 🔺 What Is Privilege Escalation?

**Privilege Escalation (PrivEsc)** is when a user gains higher permissions than they are supposed to have.

In Linux, this usually means:

* From normal user ➜ root
* From limited service account ➜ full system control

There are two types:

1. **Vertical Escalation** → user ➜ root
2. **Horizontal Escalation** → userA ➜ userB

---

# 🧠 Common Linux Privilege Escalation Vectors

## 1️⃣ SUID / SGID Binaries

### What is SUID?

SUID allows a file to run with the **owner's privileges**, not the user's.

Example:

```
-rwsr-xr-x 1 root root 12345 file
```

The `s` means SUID is set.

### Why it’s dangerous

If a vulnerable SUID binary exists, it can be abused to gain root.

### Ethical Hacker Action

Look for:

```bash
find / -perm -4000 2>/dev/null
```

### Prevention

* Remove unnecessary SUID permissions
* Monitor SUID files regularly
* Use file integrity monitoring

---

## 2️⃣ Weak File Permissions

Misconfigured permissions can allow:

* Editing `/etc/passwd`
* Modifying scripts executed by root
* Writing to cron jobs

### Dangerous Example

If a root cron job runs a script:

```
/home/user/script.sh
```

And the user can modify it → root access possible.

### Prevention

* Use principle of least privilege
* Restrict write permissions
* Regular permission audits

---

## 3️⃣ Cron Jobs

Root cron jobs running writable scripts are common escalation paths.

Check:

```bash
crontab -l
ls -la /etc/cron*
```

If a script executed by root is writable → escalation possible.

---

## 4️⃣ PATH Hijacking

If a root script runs a command without full path:

Example:

```
backup.sh:
tar -czf backup.tar.gz /home
```

If PATH is manipulated, attacker may replace `tar` with malicious version.

### Prevention

* Use absolute paths in scripts:

```
/bin/tar
```

* Restrict writable directories in PATH

---

## 5️⃣ Kernel Exploits

Outdated kernels may contain vulnerabilities allowing root access.

Examples:

* Dirty COW
* OverlayFS bugs
* Polkit (PwnKit)

⚠️ These are powerful and should only be tested in lab environments.

### Prevention

* Keep system updated
* Patch regularly
* Use LTS kernels

---

## 6️⃣ Sudo Misconfigurations

Check:

```bash
sudo -l
```

If user can run certain commands as root without password, that may be exploitable.

Example:

```
User may run: vim as root
```

Certain programs can spawn shells.

### Prevention

* Avoid giving sudo rights to interactive programs
* Use command restrictions carefully
* Enable sudo logging

---

## 7️⃣ Writable /etc/passwd

If writable, attacker could add root-level user.

Check:

```bash
ls -l /etc/passwd
```

Should be:

```
-rw-r--r--
```

---

## 8️⃣ Capabilities (Linux Capabilities Abuse)

Modern Linux uses capabilities instead of full root privileges.

Check:

```bash
getcap -r / 2>/dev/null
```

Some binaries with dangerous capabilities can be abused.

---

# 🔍 Enumeration Tools (Ethical Use Only)

These tools help identify misconfigurations:

* **LinPEAS**
* **Linux Exploit Suggester**
* **LinEnum**
* **pspy**
* **GTFOBins** (reference site for allowed binary behavior)

⚠️ Only use on systems you are authorized to test.

---

# 🛡️ Defensive Best Practices

To prevent privilege escalation:

### ✔ Apply Least Privilege

* Users should not have unnecessary access

### ✔ Patch Regularly

* Especially kernel and sudo

### ✔ Monitor Logs

```
/var/log/auth.log
```

### ✔ Audit SUID Files

```
find / -perm -4000
```

### ✔ Use SELinux / AppArmor

Mandatory access control reduces escalation impact.

---

# 🧪 How to Practice Safely

I recommend:

* Hack The Box (Linux PrivEsc machines)
* TryHackMe Linux PrivEsc rooms
* VulnHub VMs
* Set up your own lab with:

  * Kali Linux
  * Metasploitable
  * Ubuntu vulnerable VM

---

# 🔥 Real-World Insight (2026 Trend)

Recent privilege escalation trends:

* Polkit vulnerabilities
* Container escape attacks
* Misconfigured Docker environments
* Kubernetes RBAC privilege abuse
* Cloud-based Linux privilege misconfigs

Modern environments often escalate via:

* Misconfigured Docker socket
* Writable container mounts
* CI/CD pipeline misconfigurations
