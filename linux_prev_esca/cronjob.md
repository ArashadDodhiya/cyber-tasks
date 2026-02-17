# 🕒 PART 1: What is a Cron Job?

## Simple Meaning:

A **cron job** is a task that runs automatically at scheduled times.

Think of it like:

> ⏰ Alarm clock for Linux commands.

Example:

* Backup every day at midnight
* Delete temp files every hour
* Run monitoring script every 5 minutes

Linux uses a service called:

```
cron
```

---

# 🧠 PART 2: How Cron Jobs Work

Cron jobs are defined in places like:

```
/etc/crontab
/etc/cron.d/
/var/spool/cron/
/etc/cron.hourly/
/etc/cron.daily/
```

---

## Example `/etc/crontab` Entry

```
* * * * * root /usr/local/bin/backup.sh
```

Let’s break it:

```
* * * * *  root   command
| | | | |
| | | | └── Day of week
| | | └──── Month
| | └────── Day of month
| └──────── Hour
└────────── Minute
```

So:

```
* * * * *
```

Means:

👉 Run every minute.

And:

```
root /usr/local/bin/backup.sh
```

Means:

👉 Run backup.sh as root.

---

# 🔥 PART 3: How Cron Jobs Lead to Privilege Escalation

Cron jobs become dangerous when:

1. They run as **root**
2. The script is **writable by normal user**
3. The script uses **relative paths**
4. The directory is writable
5. PATH hijacking is possible

Let’s see examples.

---

# 🧪 Example 1: Writable Cron Script (Very Common in CTFs)

Cron entry:

```
* * * * * root /opt/backup.sh
```

Check permissions:

```
ls -l /opt/backup.sh
```

Output:

```
-rwxrwxrwx 1 root root backup.sh
```

🚨 World writable!

That means any user can modify it.

---

## What attacker does (lab scenario)

Edit the script:

```bash
echo "/bin/bash" >> /opt/backup.sh
```

Wait 1 minute…

Cron runs it as root.

Now attacker gets:

```
root shell
```

---

# 🧪 Example 2: PATH Hijacking in Cron

Cron entry:

```
* * * * * root backup.sh
```

Inside `backup.sh`:

```bash
tar -czf backup.tar.gz /home
```

Notice:

It uses `tar` without full path.

If attacker can:

* Write to a directory in PATH
* Place fake `tar`

Then root executes attacker’s fake tar.

Boom → PrivEsc.

---

# 🧪 Example 3: Writable Cron Directory

Check:

```
ls -la /etc/cron.daily/
```

If user can write inside it:

They can drop malicious script:

```bash
nano /etc/cron.daily/rootme
```

Add:

```bash
#!/bin/bash
/bin/bash
```

Make executable:

```
chmod +x rootme
```

When cron runs daily → root shell.

---

# 🧠 Why Cron PrivEsc Works

Because:

Cron runs automatically
Cron often runs as root
If attacker can control what cron runs → attacker controls root execution

It’s not hacking cron.

It’s abusing misconfiguration.

---

# 🔎 PART 4: How to Enumerate Cron Jobs (PrivEsc Checklist)

When you get shell, check:

```
crontab -l
cat /etc/crontab
ls -la /etc/cron*
```

Then ask:

1. Is it running as root?
2. Is the script writable?
3. Is directory writable?
4. Does script use full paths?
5. Can I modify something it calls?

---

# 🛡️ PART 5: How to Prevent Cron PrivEsc

Admins should:

✔️ Use full paths in scripts:

```
/bin/tar
```

✔️ Restrict permissions:

```
chmod 700 backup.sh
```

✔️ Avoid world-writable directories
✔️ Audit cron jobs regularly

---

# 🎯 Real-World vs CTF

In CTF:

* Writable cron scripts are common

In real enterprise:

* Less common
* But still happens due to bad DevOps scripting

Most real-world cases involve:

* Weak file permissions
* Automation scripts
* Backup scripts
* Monitoring scripts

---

# 🏆 Simple Summary

Cron job PrivEsc happens when:

1. Root runs scheduled script
2. Attacker can modify that script
3. Script executes attacker’s code
4. Attacker gets root

---

# 🔥 Important Exam Tip (OSCP Style)

When you see:

```
pspy
```

You can monitor cron jobs in real-time and see what root executes.

This helps detect hidden scheduled tasks.

---

If you want next, I can explain:

* Cron + SUID combined attack
* Cron + PATH hijacking full scenario
* Real OSCP-style walkthrough
* Blue team detection techniques

What level are you preparing for? 🔐💻
