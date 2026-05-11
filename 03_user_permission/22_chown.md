# 🛠️ `chown` — Change File Owner & Group | Linux Master Note

> **Control file ownership. In Linux, every file belongs to a user and a group — `chown` lets you reassign both.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--ownership-model)
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

### What is `chown`?
`chown` (**Ch**ange **Own**ership) changes the **owner** and/or **group** of files and directories. It's essential for **multi-user systems**, **web servers**, and **security hardening**.

### Why it matters:
- **Web servers** — Apache/Nginx need correct ownership (`www-data`)
- **Shared environments** — Team collaboration requires proper group ownership
- **Security** — Wrong ownership = unauthorized access to sensitive files
- **Docker/containers** — UID mapping issues are solved with `chown`

### Key Rule:
> ⚠️ Only **root** can change file ownership. Regular users can only change group to a group they belong to.

---

## 📖 Theory — Ownership Model

### Every file has TWO ownership attributes:
```bash
$ ls -la file.txt
-rw-r--r-- 1 dipro developers 4096 May 11 14:00 file.txt
              ├───┤ ├────────┤
              Owner   Group
```

### How ownership is determined:
1. **New file creation** → Owner = creator's UID, Group = creator's primary GID
2. **In SGID directory** → Group = directory's group (inherited)
3. **Copied files** → Owner = person who copied

### UID and GID:
```bash
$ id dipro
uid=1000(dipro) gid=1000(dipro) groups=1000(dipro),27(sudo),33(www-data)

# Files store UID/GID numbers, not names
$ stat -c "%u %g" file.txt
1000 1000
```

---

## 🧰 Syntax & Options

```bash
chown [OPTIONS] [OWNER][:GROUP] FILE(s)...
```

### Ownership format:
| Format | Meaning |
|--------|---------|
| `chown user file` | Change owner only |
| `chown user:group file` | Change owner AND group |
| `chown :group file` | Change group only (same as `chgrp`) |
| `chown user: file` | Change owner + set group to user's login group |

### Options:
| Flag | Description |
|------|-------------|
| `-R` | Recursive — apply to all contents |
| `-v` | Verbose — show each change |
| `-c` | Report only when changes made |
| `-f` | Suppress error messages |
| `--reference=RFILE` | Copy ownership from reference file |
| `--from=OWNER:GROUP` | Change only if current owner/group matches |
| `-h` | Change symlink itself (not target) |
| `--preserve-root` | Don't recursively process `/` |

---

## 🟢 Basic Usage

```bash
# Change owner
$ sudo chown dipro file.txt

# Change owner and group
$ sudo chown dipro:developers file.txt

# Change group only
$ sudo chown :developers file.txt
$ sudo chgrp developers file.txt     # equivalent

# Change owner, set group to user's login group
$ sudo chown dipro: file.txt

# Check result
$ ls -la file.txt
-rw-r--r-- 1 dipro developers 4096 May 11 file.txt
```

---

## 🟡 Intermediate Usage

### Recursive ownership change
```bash
# Change entire project ownership
$ sudo chown -R dipro:developers /home/dipro/project/

# Web server document root
$ sudo chown -R www-data:www-data /var/www/html/

# Verbose output
$ sudo chown -Rv dipro:dipro ~/Documents/
changed ownership of '/home/dipro/Documents/notes.txt' to dipro:dipro
```

### Conditional ownership change
```bash
# Only change files currently owned by 'olduser'
$ sudo chown --from=olduser newuser /var/data/*

# Only change files with specific group
$ sudo chown --from=:oldgroup :newgroup /shared/*
```

### Match another file's ownership
```bash
$ sudo chown --reference=/var/www/html/index.html newfile.html
```

### Symlink handling
```bash
# Change the symlink itself (not the target)
$ sudo chown -h dipro:dipro symlink_file

# Default: changes the TARGET file
$ sudo chown dipro:dipro symlink_file
```

---

## 🔴 Advanced Usage

### Web Server Setup (Apache/Nginx) 🌐
```bash
# Standard web server ownership
$ sudo chown -R www-data:www-data /var/www/html/
$ sudo chmod -R 755 /var/www/html/

# Allow developer to edit + web server to serve
$ sudo chown -R dipro:www-data /var/www/html/
$ sudo chmod -R 775 /var/www/html/
$ sudo chmod g+s /var/www/html/     # SGID: new files inherit www-data group
```

### Docker Container UID Fix 🐳
```bash
# Container writes files as root (UID 0) — fix on host:
$ sudo chown -R $(id -u):$(id -g) ./docker-data/

# Or in Dockerfile:
# RUN chown -R 1000:1000 /app/data
```

### User Migration
```bash
# User 'alice' leaving → transfer all files to 'bob'
$ sudo find / -user alice -exec chown bob:bob {} + 2>/dev/null

# Transfer by UID (after user deletion)
$ sudo find / -uid 1005 -exec chown bob:bob {} + 2>/dev/null
```

### Security Auditing 🔒
```bash
# Find files owned by root that shouldn't be
$ find /home -user root -type f 2>/dev/null

# Find files with no owner (orphaned — possible backdoor)
$ find / -nouser 2>/dev/null
$ find / -nogroup 2>/dev/null

# Find files owned by non-system users in /etc
$ find /etc -not -user root -type f 2>/dev/null
```

### Preserve ownership during backup
```bash
# tar preserves ownership
$ sudo tar -czvf backup.tar.gz --owner=root --group=root /var/www/

# rsync preserves ownership
$ sudo rsync -av --chown=www-data:www-data /source/ /dest/
```

---

## 🔗 Piping & Combining

```bash
# Find and change ownership of specific files
$ find /var/www -name "*.php" -exec sudo chown www-data:www-data {} +

# Fix ownership for all files by a user
$ find /shared -user oldadmin | xargs sudo chown newadmin:staff

# Change ownership of recently modified files
$ find /var/log -mtime -1 -exec sudo chown syslog:adm {} +

# Combined with chmod for web deployment
$ sudo chown -R www-data:www-data /var/www/html/ && \
  sudo find /var/www/html -type d -exec chmod 755 {} + && \
  sudo find /var/www/html -type f -exec chmod 644 {} +
```

---

## 💡 Real World Pro Tips

### Tip 1: Web deployment ownership pattern
```bash
# Developer owns, web server group can read/execute
$ sudo chown -R dipro:www-data /var/www/html/
$ sudo find /var/www/html -type d -exec chmod 2755 {} +
$ sudo find /var/www/html -type f -exec chmod 644 {} +
# SGID ensures new files inherit www-data group
```

### Tip 2: Quick check ownership
```bash
$ stat -c "%U:%G" file.txt
dipro:developers

$ namei -l /var/www/html/index.html    # Show ownership for ENTIRE path
```

### Tip 3: `chgrp` shortcut
```bash
# These are equivalent:
$ sudo chown :developers file.txt
$ sudo chgrp developers file.txt
```

### Tip 4: After extracting archives
```bash
# Archives may contain wrong ownership
$ tar -xzf backup.tar.gz
$ sudo chown -R $(whoami):$(whoami) extracted_dir/
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Essential for multi-user security | Requires root for owner changes |
| Recursive mode for bulk changes | Can break services if wrong |
| Conditional changes with `--from` | No built-in undo |
| Reference file mode | Doesn't change inside archives |
| Works with symlinks (`-h`) | UID/GID confusion with containers |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Web server files | `chown www-data:www-data` | Server process needs ownership |
| Team shared directory | `chown :teamgroup` | Group collaboration |
| After git clone | `chown -R user:user` | Fix ownership |
| Docker volume data | `chown $(id -u):$(id -g)` | Map container UIDs |
| User migration | `find -user old -exec chown new` | Transfer all files |
| Security audit | `find / -nouser` | Find orphaned files |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `sudo` | Only root can change owner |
| Wrong web server ownership | Use `www-data:www-data` for Apache/Nginx |
| Changing `/` recursively | Use `--preserve-root` |
| Not fixing after archive extraction | `chown -R $(whoami) extracted/` |
| Ignoring orphaned files | Regular audit: `find / -nouser` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Check ownership of `/etc/passwd` with `ls -la`
2. Create a file and change its owner
3. Change only the group of a file

### 🟡 Intermediate
4. Set up web server ownership for `/var/www/html`
5. Use `--reference` to copy ownership
6. Use `--from` for conditional ownership change

### 🔴 Advanced
7. Migrate all files from one user to another
8. Set up developer + web server ownership with SGID
9. Write a security audit script for orphaned files

---

## 🧠 Cheat Sheet

```
chown user file          → Change owner
chown user:group file    → Change owner + group
chown :group file        → Change group only
chown user: file         → Owner + user's default group
chown -R user:group dir/ → Recursive

sudo chown www-data:www-data /var/www/html/  → Web server
sudo chown --from=old new file               → Conditional
sudo chown --reference=ref file              → Copy ownership
sudo chown -h user:group symlink             → Symlink itself

find / -nouser 2>/dev/null   → Orphaned files
stat -c "%U:%G" file         → Quick ownership check
```

---

> **Previous**: [`chmod` ←](./21_chmod.md) | **Next**: [`useradd/adduser` →](./23_useradd_adduser.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
