Great 🔥 — this is one of the **most important Linux privilege escalation topics**:

> **Abusing Binaries & Sudo Misconfigurations**

I’ll explain this in **simple terms**, step-by-step, with clear examples — the way you’d see it in a CTF or lab.

⚠️ Everything here is for learning in labs / authorized pentests only.

---

# 🧠 PART 1: Understanding Sudo

## What is sudo?

`sudo` allows a normal user to run commands as another user (usually root).

Example:

```bash
sudo apt update
```

It runs with **root privileges**.

---

# 🔎 Step 1: Always Check Sudo First

Run:

```bash
sudo -l
```

Example output:

```
User john may run the following commands:
    (ALL) NOPASSWD: /usr/bin/vim
```

This means:

* User `john`
* Can run `/usr/bin/vim`
* As root
* Without password

🚨 That’s a privilege escalation opportunity.

---

# 🎯 PART 2: Abusing Sudo with Binaries

Sometimes administrators allow users to run certain programs with sudo, thinking they’re harmless.

But many binaries can execute commands or spawn shells.

This is where abuse happens.

---

# 🔥 Example 1: Abusing `vim`

Suppose:

```bash
sudo -l
```

Shows:

```
(ALL) NOPASSWD: /usr/bin/vim
```

Run:

```bash
sudo vim
```

Inside vim, type:

```
:!bash
```

Now you get:

```
root@machine#
```

🎉 You are root.

Why?

Because vim can execute shell commands.

---

# 🔥 Example 2: Abusing `find`

If sudo allows:

```
(ALL) NOPASSWD: /usr/bin/find
```

You can use find to execute a shell.

Conceptually:

```
sudo find . -exec /bin/bash \;
```

Now you have a root shell.

Why?

Because `find` can execute other commands using `-exec`.

---

# 🔥 Example 3: Abusing `less`

If allowed:

```
sudo less /etc/profile
```

Inside less:

Type:

```
!bash
```

Boom → root shell.

Many people don’t realize `less` can execute commands.

---

# 🔥 Example 4: Abusing `tar`

If sudo allows tar:

Tar has checkpoint features that can execute commands.

Admins often think:

“Tar is just for compression.”

But tar can execute shell commands.

That’s why checking GTFOBins is critical.

---

# 🧠 Why This Works

Because:

* Sudo runs the binary as root.
* The binary itself has features to execute commands.
* If you can run shell from inside it → you get root.

It’s not a bug.
It’s misuse of permissions.

---

# 📘 What is GTFOBins?

GTFOBins is a database of Linux binaries that can be abused.

When you see a binary allowed in sudo:

1. Go to GTFOBins
2. Search the binary
3. Check “Sudo” section
4. See if it can spawn shell

This is standard OSCP methodology.

---

# 🧨 PART 3: Abusing SUID Binaries

SUID = Set User ID bit.

Check with:

```bash
find / -perm -4000 2>/dev/null
```

If a binary is:

```
-rwsr-xr-x 1 root root 54256 somebinary
```

It runs as root even if executed by normal user.

---

## 🔥 Example: SUID `bash`

If you find:

```
/bin/bash
```

With SUID bit set (rare but common in CTFs)

You can run:

```
bash -p
```

Now you get root.

---

## 🔥 Example: Custom SUID Binary

Imagine:

```
/opt/backup
```

Owned by root with SUID.

If it calls system commands without full path:

```c
system("tar -czf backup.tar.gz /home");
```

You may combine:

* SUID abuse
* PATH hijacking

And escalate.

---

# 🧠 PART 4: Dangerous Sudo Misconfigurations

Here are common real-world misconfigs:

---

## 🚨 1. NOPASSWD Misuse

```
(ALL) NOPASSWD: ALL
```

User can run anything as root.

That’s basically root access.

---

## 🚨 2. Wildcard Abuse

Example:

```
john ALL=(root) /bin/cp *
```

Wildcards can sometimes be abused to copy protected files.

---

## 🚨 3. Running Scripts as Root

If sudo allows:

```
(ALL) NOPASSWD: /home/john/script.sh
```

And user can edit that script…

They can insert:

```bash
/bin/bash
```

Then:

```
sudo /home/john/script.sh
```

→ root shell.

---

# 🏆 Real-World Most Common PrivEsc

In real pentests, 70%+ of Linux priv esc comes from:

✔️ Misconfigured sudo
✔️ Writable root scripts
✔️ SUID misconfigurations
✔️ Credentials in files

Not kernel exploits.

---

# 🛡️ How to Defend

Admins should:

* Avoid giving shell-capable binaries in sudo
* Use full command restrictions
* Avoid wildcards
* Remove NOPASSWD unless necessary
* Regularly audit sudoers file:

```bash
cat /etc/sudoers
```

---

# 🎯 Simple Summary

Abusing binaries works when:

1. You can run a binary as root
2. That binary can execute commands
3. You use that feature to spawn a shell

It’s not “hacking the binary”
It’s abusing its features with elevated privileges.

---

If you want, next I can explain:

* Real OSCP-style priv esc scenario step-by-step
* How to think during CTF priv esc
* Advanced sudo tricks
* How defenders detect sudo abuse

What level are you preparing for? 🔐💻
