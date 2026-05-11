# 🛠️ `useradd` & `adduser` — Create New Users | Linux Master Note

> **Control who has access to your system. User management is the FOUNDATION of Linux security.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--user-account-system)
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

### `useradd` vs `adduser` — What's the difference?

| Feature | `useradd` | `adduser` |
|---------|-----------|-----------|
| Type | **Low-level binary** | **Perl/bash wrapper script** |
| Home dir | ❌ Not created by default | ✅ Creates automatically |
| Password | ❌ Must set separately | ✅ Prompts interactively |
| Shell | Uses system default | Sets `/bin/bash` |
| Availability | All Linux distros | Debian/Ubuntu (may not exist on RHEL) |
| Automation | ✅ Great for scripts | ❌ Interactive — bad for scripts |

### Key Rule:
> 🎯 Use `adduser` for **interactive** user creation. Use `useradd` for **scripts and automation**.

---

## 📖 Theory — User Account System

### Where user data is stored:

| File | Purpose | Example Entry |
|------|---------|---------------|
| `/etc/passwd` | User account info | `dipro:x:1000:1000:Dipro Debnath:/home/dipro:/bin/bash` |
| `/etc/shadow` | Encrypted passwords | `dipro:$6$xyz...:19855:0:99999:7:::` |
| `/etc/group` | Group definitions | `developers:x:1001:dipro,alice` |
| `/etc/gshadow` | Group passwords | `developers:!::dipro,alice` |

### `/etc/passwd` fields:
```
dipro : x : 1000 : 1000 : Dipro Debnath : /home/dipro : /bin/bash
  │     │    │       │         │              │            │
  │     │    │       │         │              │            └─ Login shell
  │     │    │       │         │              └─ Home directory
  │     │    │       │         └─ GECOS (full name/comment)
  │     │    │       └─ Primary GID
  │     │    └─ UID
  │     └─ Password placeholder (actual in /etc/shadow)
  └─ Username
```

### UID Ranges:
| Range | Type | Examples |
|-------|------|---------|
| `0` | Root | `root` |
| `1-999` | System accounts | `www-data`, `mysql`, `sshd` |
| `1000+` | Regular users | `dipro`, `alice` |
| `65534` | Nobody | `nobody` (unprivileged) |

### User creation process (internally):
1. Check if username/UID is unique
2. Add entry to `/etc/passwd`
3. Add entry to `/etc/shadow`
4. Create primary group in `/etc/group`
5. Create home directory (if `-m`)
6. Copy skeleton files from `/etc/skel/`
7. Set ownership on home directory

---

## 🧰 Syntax & Options

### `useradd` (low-level)
```bash
useradd [OPTIONS] USERNAME
```

| Flag | Description |
|------|-------------|
| `-m` | Create home directory |
| `-M` | Do NOT create home directory |
| `-d /path` | Specify custom home directory |
| `-s /bin/bash` | Set login shell |
| `-g GROUP` | Set primary group |
| `-G grp1,grp2` | Set supplementary groups |
| `-u UID` | Specify UID |
| `-c "Comment"` | Set GECOS (full name) |
| `-e YYYY-MM-DD` | Account expiration date |
| `-f DAYS` | Days after password expires until account disabled |
| `-r` | Create system account (UID < 1000, no home) |
| `-k /path` | Specify skeleton directory |
| `-p 'ENCRYPTED'` | Set encrypted password (NOT plaintext!) |
| `-D` | Show/change defaults |

### `adduser` (interactive wrapper — Debian/Ubuntu)
```bash
adduser [OPTIONS] USERNAME
```

### Related commands:
| Command | Purpose |
|---------|---------|
| `userdel` | Delete user |
| `usermod` | Modify existing user |
| `id` | Show user's UID, GID, groups |
| `whoami` | Show current username |
| `who` / `w` | Show logged-in users |

---

## 🟢 Basic Usage

### Using `adduser` (interactive — Debian/Ubuntu)
```bash
$ sudo adduser dipro
Adding user 'dipro' ...
Adding new group 'dipro' (1001) ...
Adding new user 'dipro' (1001) with group 'dipro' ...
Creating home directory '/home/dipro' ...
Copying files from '/etc/skel' ...
New password: ********
Retype new password: ********
Full Name []: Dipro Debnath
Room Number []:
Work Phone []:
Home Phone []:
Other []:
Is the information correct? [Y/n] Y
```

### Using `useradd` (scripting — all distros)
```bash
# Create user with home directory and bash shell
$ sudo useradd -m -s /bin/bash dipro

# Set password separately
$ sudo passwd dipro

# Create user with full name
$ sudo useradd -m -s /bin/bash -c "Dipro Debnath" dipro
```

### Verify user creation
```bash
$ id dipro
uid=1001(dipro) gid=1001(dipro) groups=1001(dipro)

$ grep dipro /etc/passwd
dipro:x:1001:1001:Dipro Debnath:/home/dipro:/bin/bash

$ ls -la /home/dipro/
total 20
drwxr-xr-x 2 dipro dipro 4096 May 11 14:00 .
```

---

## 🟡 Intermediate Usage

### Create user with specific groups
```bash
# Add user to supplementary groups
$ sudo useradd -m -s /bin/bash -G sudo,docker,developers dipro

# Verify
$ id dipro
uid=1001(dipro) gid=1001(dipro) groups=1001(dipro),27(sudo),999(docker),1002(developers)
```

### Create system account (for services)
```bash
# System account: no home, no login shell, UID < 1000
$ sudo useradd -r -s /usr/sbin/nologin myservice

# Verify
$ grep myservice /etc/passwd
myservice:x:998:998::/home/myservice:/usr/sbin/nologin
```

### Create user with expiration date
```bash
# Account expires on specific date
$ sudo useradd -m -s /bin/bash -e 2026-12-31 contractor

# Check expiration
$ sudo chage -l contractor
Account expires                     : Dec 31, 2026
```

### Delete users
```bash
# Delete user (keep home directory)
$ sudo userdel alice

# Delete user AND home directory
$ sudo userdel -r alice

# Delete user, home, and force (even if logged in)
$ sudo userdel -rf alice
```

### Custom skeleton directory
```bash
# Check default skeleton
$ ls -la /etc/skel/
.bash_logout  .bashrc  .profile

# Create user with custom skeleton
$ sudo useradd -m -k /custom/skel dipro
```

---

## 🔴 Advanced Usage

### Batch User Creation Script 🔧
```bash
#!/bin/bash
# create_users.sh — Bulk user creation from CSV
# Format: username,fullname,groups

while IFS=',' read -r username fullname groups; do
    sudo useradd -m -s /bin/bash -c "$fullname" -G "$groups" "$username"
    echo "$username:TempPass123!" | sudo chpasswd
    sudo chage -d 0 "$username"     # Force password change on first login
    echo "Created: $username ($fullname)"
done < users.csv
```

### Security Hardening 🔒
```bash
# Disable user login (without deleting)
$ sudo usermod -s /usr/sbin/nologin compromised_user
$ sudo usermod -L compromised_user       # Lock account

# Restrict SSH access
$ echo "AllowUsers dipro admin" | sudo tee -a /etc/ssh/sshd_config

# Set password aging policies
$ sudo chage -M 90 -W 14 -m 7 dipro
# -M 90 = max 90 days, -W 14 = warn 14 days before, -m 7 = min 7 days between changes

# Check for accounts with no password (DANGER!)
$ sudo awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow
```

### Forensics & Auditing 🕵️
```bash
# List all users
$ cat /etc/passwd | cut -d: -f1

# List all human users (UID >= 1000)
$ awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd

# Find users with UID 0 (root-equivalent — SUSPICIOUS!)
$ awk -F: '$3 == 0 {print $1}' /etc/passwd

# Check last login times
$ lastlog

# Find users with shell access
$ grep -v '/nologin\|/false' /etc/passwd
```

---

## 🔗 Piping & Combining

```bash
# Create user and immediately add to sudo
$ sudo useradd -m -s /bin/bash newuser && sudo usermod -aG sudo newuser

# List all users with their groups
$ for user in $(cut -d: -f1 /etc/passwd); do echo "$user: $(groups $user)"; done

# Find and lock all inactive users
$ lastlog -b 90 | awk 'NR>1 {print $1}' | while read user; do
    sudo usermod -L "$user"
    echo "Locked: $user"
done
```

---

## 💡 Real World Pro Tips

### Tip 1: Always use `-m` with `useradd`!
```bash
# ❌ No home directory created
$ sudo useradd dipro

# ✅ Home directory created
$ sudo useradd -m -s /bin/bash dipro
```

### Tip 2: Set password in scripts safely
```bash
# Safe way (reads from stdin)
$ echo "dipro:MySecurePass123!" | sudo chpasswd

# Force password change on first login
$ sudo chage -d 0 dipro
```

### Tip 3: Use nologin for service accounts
```bash
$ sudo useradd -r -s /usr/sbin/nologin nginx
# Service can own files but NO one can login as this user
```

### Tip 4: Check defaults
```bash
$ useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/sh
SKEL=/etc/skel
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| `adduser` is beginner-friendly | `adduser` not on all distros |
| `useradd` great for automation | `useradd` needs manual steps |
| Full control over UID/GID | Password must be set separately |
| System accounts for services | No built-in bulk creation |
| Expiration date support | Deleted user's files remain |

---

## 📍 Where & When to Use

| Scenario | Use | Why |
|----------|-----|-----|
| Interactive new user | `adduser` | Prompts for everything |
| Script automation | `useradd -m -s /bin/bash` | Non-interactive |
| Service account | `useradd -r -s /usr/sbin/nologin` | No login needed |
| Temporary contractor | `useradd -e DATE` | Auto-expires |
| Bulk creation | `useradd` in loop | Scriptable |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| `useradd` without `-m` | Always use `-m` for home directory |
| Not setting password | Run `passwd username` after creation |
| Not setting shell | Use `-s /bin/bash` |
| Giving service accounts a shell | Use `/usr/sbin/nologin` |
| Not checking for UID 0 accounts | Audit: `awk -F: '$3==0' /etc/passwd` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Create a user with `adduser` and verify with `id`
2. Create a user with `useradd -m -s /bin/bash` and set password
3. Check `/etc/passwd` and `/etc/shadow` for the new user

### 🟡 Intermediate
4. Create a user with supplementary groups (sudo, docker)
5. Create a system service account with nologin
6. Set account expiration and password aging policies

### 🔴 Advanced
7. Write a batch user creation script from CSV
8. Audit the system for UID 0 accounts and users with no password
9. Set up SSH restrictions with AllowUsers

---

## 🧠 Cheat Sheet

```
INTERACTIVE:
  sudo adduser username              → Full interactive setup

SCRIPTING:
  sudo useradd -m -s /bin/bash user  → Create with home + bash
  sudo useradd -m -G sudo,docker u   → With groups
  sudo useradd -r -s /usr/sbin/nologin svc  → Service account
  sudo useradd -e 2026-12-31 temp    → Expiring account

PASSWORD:
  sudo passwd username               → Set password
  echo "user:pass" | sudo chpasswd   → Script-safe
  sudo chage -d 0 user               → Force change on login

DELETE:
  sudo userdel username              → Keep home
  sudo userdel -r username           → Remove home too

AUDIT:
  id username                         → UID, GID, groups
  awk -F: '$3==0' /etc/passwd        → Find UID 0 (root-equiv)
  lastlog                            → Last login times
  grep -v nologin /etc/passwd        → Users with shell
```

---

> **Previous**: [`chown` ←](./22_chown.md) | **Next**: [`passwd` →](./24_passwd.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
