# 🛠️ `chmod` — Change File Permissions | Linux Master Note

> **The gatekeeper of Linux security. `chmod` controls WHO can read, write, and execute EVERY file on your system.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--linux-permission-model)
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

### What is `chmod`?
`chmod` (**Ch**ange **Mod**e) changes the **access permissions** of files and directories. It's the **#1 most important security command** in Linux.

### Why it matters:
- **Server security** — Wrong permissions = data breach
- **Web servers** — Incorrect perms = "403 Forbidden" or full compromise
- **Scripts** — Can't run without execute permission
- **CTF/Pentesting** — SUID/SGID misconfigs are **top privilege escalation vectors**

### History:
- Part of Unix since **1971** (Version 1 Unix by Ken Thompson)
- Based on Unix **discretionary access control (DAC)** model
- Governed by POSIX standards

---

## 📖 Theory — Linux Permission Model

### The 3 Permission Types:
| Symbol | Permission | For Files | For Directories |
|--------|-----------|-----------|-----------------|
| `r` (4) | **Read** | View contents | List contents (`ls`) |
| `w` (2) | **Write** | Modify contents | Create/delete files inside |
| `x` (1) | **Execute** | Run as program | Enter directory (`cd`) |

### The 3 User Classes:
| Symbol | Class | Description |
|--------|-------|-------------|
| `u` | **User/Owner** | The file's owner |
| `g` | **Group** | Users in the file's group |
| `o` | **Others** | Everyone else |
| `a` | **All** | All three (u+g+o) |

### How Permissions Are Displayed:
```
-rwxr-xr--  1  dipro  developers  4096  May 11 14:00  script.sh
│├──┤├──┤├──┤
│ u    g    o
└── File type: - = regular, d = directory, l = symlink
```

### Two Notation Systems:

#### 1. Symbolic Notation
| Operator | Meaning |
|----------|---------|
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exact permission |

#### 2. Octal (Numeric) Notation
| Octal | Binary | Permission |
|-------|--------|------------|
| `0` | `000` | `---` (none) |
| `1` | `001` | `--x` (execute) |
| `2` | `010` | `-w-` (write) |
| `3` | `011` | `-wx` |
| `4` | `100` | `r--` (read) |
| `5` | `101` | `r-x` |
| `6` | `110` | `rw-` |
| `7` | `111` | `rwx` (all) |

**Example: `chmod 755`** = Owner:`rwx`(7), Group:`r-x`(5), Others:`r-x`(5)

### Special Permission Bits:
| Bit | Octal | Name | Effect |
|-----|-------|------|--------|
| `s` on user | `4000` | **SUID** | File executes as the **file owner** |
| `s` on group | `2000` | **SGID** | Executes as **file group**; new files in dir inherit group |
| `t` on others | `1000` | **Sticky Bit** | Only owner can delete files in directory |

### The `umask`:
```bash
$ umask
0022          # Files: 666-022=644, Dirs: 777-022=755
```

---

## 🧰 Syntax & Options

```bash
chmod [OPTIONS] MODE FILE(s)...
```

| Flag | Description |
|------|-------------|
| `-R` | Apply recursively |
| `-v` | Verbose — show each file processed |
| `-c` | Report only when changes made |
| `-f` | Suppress error messages |
| `--reference=RFILE` | Copy permissions from reference file |
| `--preserve-root` | Don't recursively process `/` |

---

## 🟢 Basic Usage

```bash
# Make script executable
$ chmod +x script.sh

# Set exact permissions using octal
$ chmod 644 index.html        # rw-r--r--
$ chmod 755 app.sh            # rwxr-xr-x
$ chmod 700 private_key       # rwx------
$ chmod 600 .ssh/id_rsa       # rw------- (SSH key REQUIRED)

# Remove write for group and others
$ chmod go-w secret.txt
```

---

## 🟡 Intermediate Usage

### Recursive permissions
```bash
# Set directories to 755, files to 644 (web server standard)
$ find ~/project/ -type d -exec chmod 755 {} +
$ find ~/project/ -type f -exec chmod 644 {} +
```

### Symbolic combinations
```bash
$ chmod u+x,o-rwx script.sh
$ chmod u=rwx,g=rx,o= private_app
$ chmod ug+w shared_file.txt
```

### Match another file's permissions
```bash
$ chmod --reference=template.sh new_script.sh
```

### Verbose output
```bash
$ chmod -v 755 *.sh
mode of 'backup.sh' changed from 0644 (rw-r--r--) to 0755 (rwxr-xr-x)
```

---

## 🔴 Advanced Usage

### SUID — Set User ID 🔒
```bash
# File runs as FILE OWNER (not the user running it)
$ chmod 4755 /usr/bin/custom_tool
$ chmod u+s /usr/bin/custom_tool

# passwd command runs as root via SUID:
$ ls -la /usr/bin/passwd
-rwsr-xr-x 1 root root 68208 /usr/bin/passwd
```

### SGID — Set Group ID
```bash
# New files inherit directory's group
$ chmod 2775 /shared/teamwork/
$ chmod g+s /shared/teamwork/
```

### Sticky Bit
```bash
# Only file owners can delete their files
$ chmod 1777 /tmp/
$ chmod +t /shared/uploads/
```

### Security Auditing 🔒
```bash
# Find ALL SUID files (privilege escalation check!)
$ find / -perm -4000 -type f 2>/dev/null

# Find world-writable files
$ find / -perm -o+w -type f 2>/dev/null

# Find world-writable dirs without sticky bit (DANGEROUS)
$ find / -perm -o+w ! -perm -1000 -type d 2>/dev/null

# Find files with no owner (orphaned — possible backdoor)
$ find / -nouser -o -nogroup 2>/dev/null
```

### CTF/Pentesting — SUID Exploitation
```bash
# Find SUID binaries → Check GTFOBins
$ find / -perm -4000 -type f 2>/dev/null

# If 'find' has SUID:
$ find . -exec /bin/sh -p \; -quit    # root shell! 🎯

# If 'python3' has SUID:
$ python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

---

## 🔗 Piping & Combining

```bash
# Fix web server permissions
$ find /var/www -type d | xargs chmod 755
$ find /var/www -type f | xargs chmod 644

# Backup permissions before changing
$ getfacl -R /var/www > permissions_backup.acl
$ setfacl --restore=permissions_backup.acl

# Find executable files
$ find . -perm -u+x -type f
```

---

## 💡 Real World Pro Tips

### Tip 1: SSH Key Permissions (REQUIRED or SSH refuses!)
```bash
$ chmod 700 ~/.ssh/
$ chmod 600 ~/.ssh/id_rsa
$ chmod 644 ~/.ssh/id_rsa.pub
$ chmod 600 ~/.ssh/authorized_keys
```

### Tip 2: Never use 777!
```bash
# ❌ NEVER — total compromise risk
$ chmod 777 /var/www/html/

# ✅ CORRECT
$ chmod 755 /var/www/html/
$ chown www-data:www-data /var/www/html/
```

### Tip 3: Use `stat` for detailed info
```bash
$ stat -c "%a %A %U:%G %n" /etc/passwd
644 -rw-r--r-- root:root /etc/passwd
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Fundamental security control | Octal notation confusing for beginners |
| Two notation systems | No undo — wrong chmod breaks system |
| Recursive mode for bulk | SUID misconfig = privilege escalation |
| Universal Unix/Linux | Doesn't handle ACLs (need `setfacl`) |

---

## 📍 Where & When to Use

| Scenario | Recommended | Why |
|----------|-------------|-----|
| Regular file | `644` | Owner rw, others read |
| Shell script | `755` | Owner rwx, others run |
| SSH private key | `600` | Owner only |
| Web directory | `755` | Web server reads |
| Shared team dir | `2775` (SGID) | Inherit group |
| `/tmp` directory | `1777` (sticky) | Protect deletion |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| `chmod 777` on anything | Use `755`/`644` appropriately |
| `chmod -R 755` on all files | Separate: dirs=755, files=644 |
| Wrong SSH key perms | `chmod 600 ~/.ssh/id_rsa` |
| Not knowing SUID risks | Audit with `find / -perm -4000` |
| `chmod` on `/` recursively | ALWAYS use `--preserve-root` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Create a file and check default permissions with `ls -la`
2. Make a script executable with `chmod +x`
3. Set a file to `644` and explain what each digit means

### 🟡 Intermediate
4. Set up proper SSH directory permissions
5. Use `find` + `chmod` to set dirs=755, files=644
6. Create a shared directory with SGID

### 🔴 Advanced
7. Find all SUID binaries and check GTFOBins
8. Create a security audit script for world-writable files
9. Exploit a misconfigured SUID binary (in a lab)

---

## 🧠 Cheat Sheet

```
r=4  w=2  x=1  |  SUID=4000  SGID=2000  Sticky=1000

chmod 644 file   → rw-r--r--    chmod u+x file   → Add execute
chmod 755 file   → rwxr-xr-x   chmod go-w file  → Remove write
chmod 700 file   → rwx------   chmod a+r file   → Add read all
chmod 600 file   → rw-------   chmod o= file    → Remove all others
chmod 4755 file  → SUID set    chmod 2775 dir   → SGID set
chmod 1777 dir   → Sticky set

find / -perm -4000 2>/dev/null  → Find SUID files
stat -c "%a %A %U:%G" file     → Detailed perms
```

---

> **Previous**: [`wc` ←](../02_text_processing/20_wc.md) | **Next**: [`chown` →](./22_chown.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
