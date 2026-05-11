# 🛠️ `passwd` — Change User Password | Linux Master Note

> **The first line of authentication defense. `passwd` manages the passwords that protect every account on the system.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--password-storage--hashing)
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

### What is `passwd`?
`passwd` changes user account passwords. It's one of the few commands that runs with **SUID** — meaning it executes as **root** even when run by a regular user, so it can write to `/etc/shadow`.

### Why it matters:
- **Authentication** — Passwords are the primary login mechanism
- **Security** — Weak passwords = easy brute force
- **Compliance** — Password aging policies are required by most security standards
- **Incident Response** — Locking/unlocking accounts during security incidents

### Key Fact:
```bash
$ ls -la /usr/bin/passwd
-rwsr-xr-x 1 root root 68208 /usr/bin/passwd
#   ^--- SUID bit! Runs as root
```

---

## 📖 Theory — Password Storage & Hashing

### Where passwords live:

| File | Permissions | Purpose |
|------|------------|---------|
| `/etc/passwd` | `644` (world-readable) | User info — **NO passwords** (just `x` placeholder) |
| `/etc/shadow` | `640` (root only) | **Encrypted password hashes** |

### `/etc/shadow` format:
```
dipro:$6$rounds=5000$salt$hash:19855:0:99999:7:::
  │     │                        │    │   │    │
  │     │                        │    │   │    └─ Warning days before expire
  │     │                        │    │   └─ Max days (password must change)
  │     │                        │    └─ Min days (between changes)
  │     │                        └─ Last change (days since Jan 1, 1970)
  │     └─ Encrypted password hash
  └─ Username
```

### Hash Algorithm Identifiers:
| Prefix | Algorithm | Security |
|--------|-----------|----------|
| `$1$` | MD5 | ❌ Weak — crackable |
| `$5$` | SHA-256 | ⚠️ Moderate |
| `$6$` | SHA-512 | ✅ Strong (Linux default) |
| `$y$` | yescrypt | ✅ Modern (newest distros) |
| `$2b$` | bcrypt | ✅ Strong (BSD/some Linux) |

### Special shadow field values:
| Value | Meaning |
|-------|---------|
| `!` or `!!` | Account locked (cannot login with password) |
| `*` | Account disabled (system accounts) |
| (empty) | **NO PASSWORD — DANGER!** Anyone can login |

### Password validation process:
1. User enters password
2. System reads hash from `/etc/shadow`
3. Extracts **salt** from stored hash
4. Hashes entered password with same salt + algorithm
5. Compares result with stored hash
6. Match → login granted; No match → denied

---

## 🧰 Syntax & Options

```bash
passwd [OPTIONS] [USERNAME]
```

| Flag | Description |
|------|-------------|
| (none) | Change current user's password |
| `-l` | **Lock** account (prepend `!` to hash) |
| `-u` | **Unlock** account (remove `!`) |
| `-d` | **Delete** password (make empty — DANGEROUS!) |
| `-e` | **Expire** password (force change on next login) |
| `-n DAYS` | Minimum days between password changes |
| `-x DAYS` | Maximum days before password must be changed |
| `-w DAYS` | Warning days before password expires |
| `-i DAYS` | Inactive days after expiry before account disabled |
| `-S` | Show password **status** |
| `--stdin` | Read password from stdin (Red Hat/CentOS) |

### Related: `chage` (password aging)
```bash
chage [OPTIONS] USERNAME
```

| Flag | Description |
|------|-------------|
| `-l` | List all aging info |
| `-d 0` | Force password change on next login |
| `-M DAYS` | Maximum password age |
| `-m DAYS` | Minimum password age |
| `-W DAYS` | Warning days |
| `-E DATE` | Account expiration date |

---

## 🟢 Basic Usage

```bash
# Change your own password
$ passwd
Changing password for dipro.
Current password: ********
New password: ********
Retype new password: ********
passwd: password updated successfully

# Change another user's password (requires root)
$ sudo passwd alice
New password: ********
Retype new password: ********
passwd: password updated successfully

# Check password status
$ sudo passwd -S dipro
dipro P 05/11/2026 0 99999 7 -1
# P = Password set, L = Locked, NP = No password
```

---

## 🟡 Intermediate Usage

### Lock/Unlock accounts
```bash
# Lock account (cannot login)
$ sudo passwd -l suspicious_user
passwd: password expiry information changed.

# Verify locked (see '!' in shadow)
$ sudo grep suspicious_user /etc/shadow
suspicious_user:!$6$...:...

# Unlock account
$ sudo passwd -u suspicious_user
```

### Force password change on next login
```bash
$ sudo passwd -e dipro
# OR
$ sudo chage -d 0 dipro

# Next time dipro logs in:
# "You are required to change your password immediately (administrator enforced)"
```

### Password aging policies
```bash
# Set password policy: min 7 days, max 90 days, warn 14 days
$ sudo passwd -n 7 -x 90 -w 14 dipro

# Or using chage (more options):
$ sudo chage -m 7 -M 90 -W 14 dipro

# View all aging info
$ sudo chage -l dipro
Last password change                    : May 11, 2026
Password expires                        : Aug 09, 2026
Password inactive                       : never
Account expires                         : never
Minimum number of days between password change  : 7
Maximum number of days between password change  : 90
Number of days of warning before password expires: 14
```

### Set password in scripts
```bash
# Method 1: chpasswd (RECOMMENDED for scripts)
$ echo "dipro:NewSecureP@ss123" | sudo chpasswd

# Method 2: batch mode
$ sudo chpasswd <<EOF
alice:Alice_Pass_2026!
bob:Bob_Secure_Pass!
EOF

# Method 3: openssl (generate hash)
$ echo "dipro:$(openssl passwd -6 'MyPassword')" | sudo chpasswd -e
```

---

## 🔴 Advanced Usage

### Security Auditing 🔒
```bash
# Find accounts with NO password (CRITICAL VULNERABILITY!)
$ sudo awk -F: '($2 == "" || $2 == " ") {print $1}' /etc/shadow

# Find locked accounts
$ sudo awk -F: '$2 ~ /^!/ {print $1}' /etc/shadow

# Find accounts with expired passwords
$ sudo awk -F: '$2 ~ /^\*/ {print $1}' /etc/shadow

# Check password age for all users
$ for user in $(awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd); do
    echo "=== $user ==="
    sudo chage -l "$user"
done

# Find accounts that never expire (possible policy violation)
$ sudo awk -F: '$5 == 99999 || $5 == "" {print $1}' /etc/shadow
```

### PAM Password Policy (system-wide) 🛡️
```bash
# Install password quality checker
$ sudo apt install libpam-pwquality

# Edit /etc/security/pwquality.conf
minlen = 12           # Minimum 12 characters
dcredit = -1          # At least 1 digit
ucredit = -1          # At least 1 uppercase
lcredit = -1          # At least 1 lowercase
ocredit = -1          # At least 1 special char
maxrepeat = 3         # Max 3 consecutive same chars
reject_username       # Can't contain username
enforce_for_root      # Applies to root too
```

### Incident Response — Mass Lock
```bash
#!/bin/bash
# lock_all_users.sh — Emergency: lock all non-root users
for user in $(awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd); do
    sudo passwd -l "$user"
    echo "LOCKED: $user"
done
echo "⚠️ All user accounts locked!"
```

### Password Cracking Awareness 🔐
```bash
# Extract hashes for offline cracking (pentesting only!)
$ sudo cat /etc/shadow | grep '^\w' > hashes.txt

# John the Ripper
$ john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Hashcat (GPU)
$ hashcat -m 1800 hashes.txt rockyou.txt    # Mode 1800 = SHA-512
```

---

## 🔗 Piping & Combining

```bash
# Set password non-interactively
$ echo "username:password" | sudo chpasswd

# Force all users to change password
$ for user in $(awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd); do
    sudo chage -d 0 "$user"
done

# Check if a password is set
$ sudo passwd -S username | awk '{print $2}'
# P = set, L = locked, NP = no password

# Generate random password
$ openssl rand -base64 16
$ tr -dc 'A-Za-z0-9!@#$%' </dev/urandom | head -c 20
```

---

## 💡 Real World Pro Tips

### Tip 1: Never use `passwd -d` in production!
```bash
# ❌ Removes password — anyone can login!
$ sudo passwd -d username

# ✅ Lock the account instead
$ sudo passwd -l username
```

### Tip 2: Enforce password change for new users
```bash
$ sudo useradd -m -s /bin/bash newuser
$ echo "newuser:TempPass123!" | sudo chpasswd
$ sudo chage -d 0 newuser     # Force change on first login
```

### Tip 3: Check your own password status
```bash
$ passwd -S
dipro P 05/11/2026 0 99999 7 -1
# Format: user status lastchange min max warn inactive
```

### Tip 4: Generate strong random passwords
```bash
# Using openssl
$ openssl rand -base64 20
K3j9F1p2R5tY8u0I4o6A7sD=

# Using /dev/urandom
$ tr -dc 'A-Za-z0-9!@#$%^&*' </dev/urandom | head -c 24; echo
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| SUID — regular users can change own password | Interactive — hard to script directly |
| Lock/unlock accounts instantly | No built-in password strength check |
| Password aging support | Doesn't enforce policy (need PAM) |
| Works on all Linux distros | Hash algorithm depends on system config |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Change own password | `passwd` | Self-service |
| Reset user password | `sudo passwd user` | Admin action |
| Lock compromised account | `sudo passwd -l user` | Incident response |
| Script password setting | `echo "u:p" \| sudo chpasswd` | Automation |
| Force password change | `sudo chage -d 0 user` | Policy compliance |
| Audit password status | `sudo passwd -S user` | Security check |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `passwd -d` | Use `passwd -l` to lock instead |
| Hardcoding passwords in scripts | Use `chpasswd` with stdin |
| No password aging policy | Set with `chage -M 90 -W 14` |
| Not forcing change for new users | `chage -d 0 newuser` |
| Ignoring accounts with no password | Audit: `awk -F: '$2==""' /etc/shadow` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Change your own password
2. Check password status with `passwd -S`
3. Create a new user and set their password

### 🟡 Intermediate
4. Lock and unlock a user account
5. Set password aging: min 7, max 90, warn 14 days
6. Force a user to change password on next login

### 🔴 Advanced
7. Write a script to set passwords for 10 users from a file
8. Audit all accounts for empty passwords
9. Configure PAM password quality requirements

---

## 🧠 Cheat Sheet

```
passwd                    → Change own password
sudo passwd user          → Change user's password
sudo passwd -l user       → Lock account
sudo passwd -u user       → Unlock account
sudo passwd -e user       → Expire (force change)
sudo passwd -S user       → Show status
sudo passwd -d user       → Delete password (DANGER!)

echo "user:pass" | sudo chpasswd     → Script-safe
sudo chage -d 0 user                 → Force change next login
sudo chage -l user                   → Show aging info
sudo chage -M 90 -W 14 user          → Set policy

AUDIT:
  awk -F: '$2==""' /etc/shadow       → No password
  awk -F: '$2~/^!/' /etc/shadow      → Locked accounts
  passwd -S user | awk '{print $2}'  → P/L/NP status
```

---

> **Previous**: [`useradd/adduser` ←](./23_useradd_adduser.md) | **Next**: [`su/sudo` →](./25_su_sudo.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
