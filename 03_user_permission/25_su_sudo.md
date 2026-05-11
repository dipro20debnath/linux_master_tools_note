# 🛠️ `su` & `sudo` — Switch User / Execute as Superuser | Linux Master Note

> **The keys to the kingdom. `su` lets you BECOME another user. `sudo` lets you RUN commands as another user. Misconfigure these = instant root compromise.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--privilege-escalation-model)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [Piping & Combining](#-piping--combining)
8. [Real World Pro Tips](#-real-world-pro-tips)
9. [Pros & Cons](#-pros--cons)
10. [Where & When to Use](#-where--when-to-use)
11. [Common Mistakes](#-common-mistakes)
12. [Practice Exercises](#-practice-exercises)
13. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### `su` vs `sudo` — Critical Difference:

| Feature | `su` | `sudo` |
|---------|------|--------|
| Meaning | **S**witch **U**ser | **S**uperuser **Do** |
| Authentication | Target user's password | YOUR own password |
| Session | Opens new shell as that user | Runs single command |
| Audit trail | ❌ Poor — just shows "su" | ✅ Logs every command |
| Granularity | All or nothing | Per-command control |
| Best practice | ⚠️ Discouraged | ✅ Recommended |

### Why `sudo` > `su`:
1. **Least privilege** — Only elevate when needed
2. **Audit logging** — Every `sudo` command is logged in `/var/log/auth.log`
3. **No root password sharing** — Users use their own password
4. **Fine-grained control** — Allow specific commands only
5. **Time-limited** — Credentials cached for ~15 minutes (configurable)

---

## 📖 Theory — Privilege Escalation Model

### How `su` works:
1. User runs `su [username]`
2. System prompts for **target user's** password
3. If correct → spawns new shell as that user
4. Inherits target user's environment (with `su -`)
5. `exit` returns to original user

### How `sudo` works:
1. User runs `sudo command`
2. System checks `/etc/sudoers` for authorization
3. Prompts for **user's own** password
4. If authorized → executes command as target user (default: root)
5. Logs the action to `/var/log/auth.log`
6. Caches credentials for ~15 minutes

### The sudoers file (`/etc/sudoers`):
```
# Format: WHO  WHERE = (AS_WHOM) WHAT
root    ALL=(ALL:ALL) ALL
%sudo   ALL=(ALL:ALL) ALL
%admin  ALL=(ALL) ALL

# Breakdown:
# root   → username
# ALL=   → from any host
# (ALL:ALL) → as any user:any group
# ALL    → can run any command
```

### Sudoers security:
```bash
# ⚠️ NEVER edit /etc/sudoers directly!
# ALWAYS use visudo — it validates syntax before saving

$ sudo visudo
```

---

## 🧰 Syntax & Options

### `su` syntax:
```bash
su [OPTIONS] [USERNAME]
```

| Flag | Description |
|------|-------------|
| `-` or `-l` or `--login` | Login shell (full environment switch) |
| `-c COMMAND` | Run single command as user |
| `-s SHELL` | Specify shell |
| `-m` or `-p` | Preserve current environment |

### `sudo` syntax:
```bash
sudo [OPTIONS] COMMAND
```

| Flag | Description |
|------|-------------|
| `-u USER` | Run as specified user (default: root) |
| `-i` | Login shell as target user (like `su -`) |
| `-s` | Run shell as target user |
| `-l` | List allowed commands for current user |
| `-k` | Invalidate cached credentials |
| `-v` | Extend timeout without running command |
| `-n` | Non-interactive (fail if password needed) |
| `-E` | Preserve environment variables |
| `-H` | Set HOME to target user's home |
| `-b` | Run command in background |
| `!!` | Re-run last command with sudo |

---

## 🟢 Basic Usage

### `su` basics:
```bash
# Switch to root (requires root password)
$ su
Password: ********
root@host:~#

# Switch to root with login shell (RECOMMENDED)
$ su -
Password: ********
root@host:~#

# Switch to another user
$ su - alice
Password: ********    # alice's password
alice@host:~$

# Run single command as another user
$ su -c "cat /etc/shadow" root
```

### `sudo` basics:
```bash
# Run command as root
$ sudo apt update

# Run command as another user
$ sudo -u www-data cat /var/www/html/config.php

# Open root shell
$ sudo -i
root@host:~#

# Check what you can run
$ sudo -l
User dipro may run the following commands on host:
    (ALL : ALL) ALL

# Re-run last command with sudo (when you forget!)
$ apt update
E: Could not open lock file...
$ sudo !!
sudo apt update       # ← Automatically prepends sudo!
```

---

## 🟡 Intermediate Usage

### Sudoers configuration
```bash
# ALWAYS use visudo!
$ sudo visudo

# Give user full sudo access
dipro   ALL=(ALL:ALL) ALL

# Give user sudo without password
dipro   ALL=(ALL) NOPASSWD: ALL

# Allow specific commands only
dipro   ALL=(ALL) /usr/bin/apt, /usr/bin/systemctl

# Allow specific commands without password
dipro   ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# Group-based sudo (recommended)
%developers ALL=(ALL) /usr/bin/docker, /usr/bin/docker-compose
%webadmins  ALL=(ALL) /usr/bin/systemctl restart nginx, /usr/bin/systemctl restart apache2
```

### Drop-in sudoers files (recommended)
```bash
# Instead of editing main sudoers, create separate files:
$ sudo visudo -f /etc/sudoers.d/dipro
dipro ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

$ sudo visudo -f /etc/sudoers.d/developers
%developers ALL=(ALL) /usr/bin/docker, /usr/bin/git
```

### `su` with preserved environment
```bash
# Full login shell (clean environment)
$ su - root

# Preserve current environment
$ su -m root

# Compare:
$ su -c 'echo $PATH' root          # root's PATH
$ su -m -c 'echo $PATH' root       # YOUR PATH kept
```

### Invalidate/refresh credentials
```bash
# Force password prompt next time
$ sudo -k

# Extend timeout without running command
$ sudo -v
```

---

## 🔴 Advanced Usage

### Sudoers Advanced Directives 🔒
```bash
# /etc/sudoers (via visudo)

# Command aliases — group commands
Cmnd_Alias SERVICES = /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *
Cmnd_Alias NETWORK = /usr/sbin/iptables, /usr/sbin/ifconfig, /usr/bin/ip

# User aliases
User_Alias ADMINS = dipro, alice, bob
User_Alias DEVOPS = charlie, dave

# Host aliases
Host_Alias WEBSERVERS = web1, web2, web3

# Apply rules
ADMINS    ALL=(ALL) ALL
DEVOPS    WEBSERVERS=(root) SERVICES, NETWORK

# Deny specific commands
dipro     ALL=(ALL) ALL, !/usr/bin/su, !/bin/bash, !/usr/bin/passwd root

# Require password for dangerous commands
dipro     ALL=(ALL) NOPASSWD: /usr/bin/apt, PASSWD: /usr/sbin/visudo

# Logging — log I/O
Defaults  log_input, log_output
Defaults  logfile="/var/log/sudo.log"

# Strict timeout
Defaults  timestamp_timeout=5    # 5 minutes (default: 15)
Defaults  passwd_tries=3         # Max 3 password attempts
```

### Security Auditing 🔒
```bash
# Check who has sudo access
$ grep -E '^[^#].*ALL' /etc/sudoers /etc/sudoers.d/* 2>/dev/null

# Check sudo group members
$ getent group sudo
sudo:x:27:dipro,alice

# View sudo log
$ sudo cat /var/log/auth.log | grep sudo

# Find ALL sudoers.d files
$ ls -la /etc/sudoers.d/

# Check for NOPASSWD (potential risk)
$ sudo grep -r "NOPASSWD" /etc/sudoers /etc/sudoers.d/
```

### CTF/Pentesting — Sudo Exploitation 🎯
```bash
# Step 1: Check what sudo commands you can run
$ sudo -l
User dipro may run the following commands:
    (ALL) NOPASSWD: /usr/bin/vim
    (ALL) NOPASSWD: /usr/bin/find
    (ALL) NOPASSWD: /usr/bin/python3

# Step 2: Check GTFOBins for each binary!

# vim → root shell
$ sudo vim -c ':!/bin/bash'

# find → root shell
$ sudo find / -exec /bin/bash \; -quit

# python3 → root shell
$ sudo python3 -c 'import os; os.system("/bin/bash")'

# less → root shell
$ sudo less /etc/hosts
!/bin/bash

# env → root shell
$ sudo env /bin/bash

# awk → root shell
$ sudo awk 'BEGIN {system("/bin/bash")}'

# Check sudo version for CVEs
$ sudo --version
# CVE-2021-3156 (Baron Samedit) — heap overflow in sudo < 1.9.5p2
```

### `su` for forensics
```bash
# Test if you know a user's password (pentesting)
$ su -c "id" targetuser

# Switch to www-data to see web server perspective
$ sudo su - www-data -s /bin/bash
```

---

## 🔗 Piping & Combining

```bash
# Run piped command as root
$ sudo sh -c 'echo "nameserver 8.8.8.8" >> /etc/resolv.conf'

# Redirect as root (sudo doesn't affect pipes!)
# ❌ WRONG — redirect runs as YOUR user
$ sudo echo "data" > /root/file.txt

# ✅ CORRECT — use tee
$ echo "data" | sudo tee /root/file.txt
$ echo "data" | sudo tee -a /root/file.txt    # append

# Run multiple commands as root
$ sudo bash -c 'apt update && apt upgrade -y'

# Switch user and run commands
$ sudo -u postgres psql -c "SELECT version();"
```

---

## 💡 Real World Pro Tips

### Tip 1: `sudo !!` — The life-saver
```bash
$ apt update
E: Could not open lock file /var/lib/dpkg/lock
$ sudo !!          # Re-runs "apt update" with sudo
```

### Tip 2: Use `sudo -i` instead of `su -`
```bash
# ❌ Requires root password (may not be set on Ubuntu)
$ su -

# ✅ Uses YOUR password, better audit trail
$ sudo -i
```

### Tip 3: Redirect output as root
```bash
# ❌ FAILS — shell redirects as YOUR user
$ sudo echo "127.0.0.1 mysite" >> /etc/hosts

# ✅ Use tee
$ echo "127.0.0.1 mysite" | sudo tee -a /etc/hosts
```

### Tip 4: Secure sudoers practices
```bash
# Always use visudo (validates syntax)
$ sudo visudo

# Use drop-in files instead of editing main file
$ sudo visudo -f /etc/sudoers.d/myconfig

# Set strict timestamp
Defaults timestamp_timeout=5
```

---

## ✅ Pros & Cons

### `su`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Full shell as target user | Requires target's password |
| Simple for root access | No audit logging |
| Works without sudoers setup | All-or-nothing access |

### `sudo`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Uses YOUR password | Complex sudoers syntax |
| Detailed audit logging | Misconfigured = security hole |
| Fine-grained control | Can be exploited via GTFOBins |
| Per-command authorization | Cached credentials can be abused |
| Industry standard | NOPASSWD overuse is dangerous |

---

## 📍 Where & When to Use

| Scenario | Use | Why |
|----------|-----|-----|
| Single root command | `sudo command` | Least privilege |
| Root shell session | `sudo -i` | Logged, your password |
| Run as another user | `sudo -u user cmd` | Controlled delegation |
| Become another user fully | `su - user` | Full environment switch |
| Allow developers Docker | sudoers rule | Specific command access |
| Incident response | `sudo -k` | Clear cached creds |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `su` instead of `sudo` | Prefer `sudo` for audit trail |
| `sudo echo "x" > /root/file` | Use `echo "x" \| sudo tee file` |
| Editing sudoers directly | ALWAYS use `visudo` |
| `NOPASSWD: ALL` for users | Limit to specific commands |
| Not checking `sudo -l` in pentests | Always enumerate sudo privileges! |
| Syntax error in sudoers = lockout | `visudo` prevents this |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Run `apt update` with `sudo`
2. Use `sudo !!` after a permission denied error
3. Open a root shell with `sudo -i`

### 🟡 Intermediate
4. Create a sudoers drop-in file for a user
5. Allow a user to restart nginx without password
6. Use `sudo -u www-data` to test web server perspective

### 🔴 Advanced
7. Configure command aliases in sudoers
8. Check `sudo -l` and exploit allowed binaries (GTFOBins)
9. Set up sudo logging with I/O recording

---

## 🧠 Cheat Sheet

```
SU:
  su               → Root shell (root password)
  su -             → Root login shell
  su - alice       → Login as alice
  su -c "cmd"      → Run single command

SUDO:
  sudo command     → Run as root
  sudo -i          → Root login shell (YOUR password)
  sudo -u user cmd → Run as specific user
  sudo -l          → List allowed commands
  sudo -k          → Clear cached credentials
  sudo !!          → Re-run last command with sudo

SUDOERS (visudo):
  user ALL=(ALL) ALL                → Full access
  user ALL=(ALL) NOPASSWD: /cmd     → Specific no-password
  %group ALL=(ALL) /cmd1, /cmd2     → Group-based
  Defaults timestamp_timeout=5      → 5-min timeout

REDIRECT AS ROOT:
  echo "data" | sudo tee file       → Write
  echo "data" | sudo tee -a file    → Append

AUDIT:
  grep sudo /var/log/auth.log       → Sudo log
  getent group sudo                 → Sudo group members
  sudo -l                           → Your permissions
```

---

> **Previous**: [`passwd` ←](./24_passwd.md) | **Next**: [`groups/usermod` →](./26_groups_usermod.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
