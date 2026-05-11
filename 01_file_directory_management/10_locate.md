# 🛠️ `locate` — Fast File Search by Name | Linux Master Note

> **Blazing fast file search using a pre-built database. Find any file in milliseconds.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory)
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

### What is `locate`?
`locate` finds files by name using a **pre-built database** (`/var/lib/mlocate/mlocate.db`). It's dramatically faster than `find` because it searches an indexed database instead of traversing the filesystem.

### `locate` vs `find`:
| Feature | `locate` | `find` |
|---------|----------|--------|
| Speed | ⚡ Milliseconds | 🐌 Seconds to minutes |
| Accuracy | May be outdated | Always current |
| Search by | Name only | Name, size, time, perms, etc. |
| Database | Required (`updatedb`) | Not needed |
| Actions | Cannot execute | Can execute (`-exec`) |

---

## 📖 Theory

### How `locate` works:
1. `updatedb` command scans the entire filesystem
2. Creates/updates a compressed database at `/var/lib/mlocate/mlocate.db`
3. `locate` searches this database using fast string matching
4. Database is typically updated daily via cron job

### Database location:
```bash
$ ls -la /var/lib/mlocate/mlocate.db
# OR (on some systems)
$ ls -la /var/lib/plocate/plocate.db
```

---

## 🧰 Syntax & Options

```bash
locate [OPTIONS] PATTERN
```

| Flag | Description |
|------|-------------|
| `-i` | Case-insensitive search |
| `-n NUM` / `--limit NUM` | Limit output to NUM results |
| `-c` / `--count` | Print only count of matches |
| `-r` | Use POSIX regex pattern |
| `--regex` | Use extended regex |
| `-e` / `--existing` | Print only existing files |
| `-b` / `--basename` | Match only the basename (not full path) |
| `-S` / `--statistics` | Database statistics |
| `-w` / `--wholename` | Match against whole path (default) |

---

## 🟢 Basic Usage

### Install locate
```bash
# Debian/Ubuntu
$ sudo apt install mlocate
# OR (newer)
$ sudo apt install plocate

# Red Hat/CentOS
$ sudo yum install mlocate

# Update the database (REQUIRED after first install!)
$ sudo updatedb
```

### Basic search
```bash
# Find all files named "passwd"
$ locate passwd
/etc/passwd
/etc/passwd-
/usr/share/doc/passwd/
/usr/share/man/man5/passwd.5.gz

# Find .conf files
$ locate "*.conf"

# Case-insensitive search
$ locate -i "readme"
```

---

## 🟡 Intermediate Usage

### Limit results
```bash
$ locate -n 5 "*.log"    # Show only first 5 results
```

### Count matches
```bash
$ locate -c "*.py"       # Count Python files
1247
```

### Search only basename
```bash
# Match filename only (not path)
$ locate -b "config.json"
# Returns: /app/config.json (not /config.json/data)
```

### Check only existing files
```bash
$ locate -e "*.txt"      # Skip files that were deleted since last updatedb
```

### Database statistics
```bash
$ locate -S
Database /var/lib/mlocate/mlocate.db:
    45,230 directories
   312,456 files
 15,678,234 bytes in file names
  6,234,567 bytes used to store database
```

---

## 🔴 Advanced Usage

### Regex search
```bash
# Find files matching regex
$ locate -r "/etc/.*\.conf$"

# Find Python or JavaScript files
$ locate --regex "\.(py|js)$"

# Find files in specific directory pattern
$ locate -r "/home/dipro/.*\.md$"
```

### Custom database
```bash
# Create a project-specific database
$ updatedb -l 0 -o /tmp/myproject.db -U /home/dipro/projects

# Search the custom database
$ locate -d /tmp/myproject.db "*.py"
```

### Combining with other tools
```bash
# Find and get detailed info
$ locate "*.conf" | head -10 | xargs ls -la

# Find and search inside files
$ locate "*.py" | xargs grep -l "import flask"

# Find and count lines
$ locate "*.md" | xargs wc -l 2>/dev/null
```

### Cron job for updatedb
```bash
# Check existing cron schedule
$ cat /etc/cron.daily/mlocate
# Runs updatedb daily — you can create custom schedules

# Manual update (run after major file changes)
$ sudo updatedb
```

### Security: What locate hides
```bash
# locate respects file permissions!
# Non-root users cannot see files they don't have access to
$ locate shadow          # Regular user: limited results
$ sudo locate shadow     # Root: shows all matches
```

---

## 🔗 Piping & Combining

```bash
# locate + grep — Filter results further
$ locate "*.log" | grep -i "error"

# locate + wc — Count results
$ locate "*.py" | wc -l

# locate + xargs + ls — Detailed file info
$ locate "nginx.conf" | xargs ls -la 2>/dev/null

# locate + xargs + grep — Search inside found files
$ locate "*.conf" | xargs grep -l "listen 80" 2>/dev/null
```

---

## 💡 Real World Pro Tips

### Tip 1: Alias for quick search
```bash
alias ff='locate -i'    # Fast find
$ ff readme              # Quick case-insensitive search
```

### Tip 2: Update database after installations
```bash
$ sudo apt install nginx
$ sudo updatedb          # Update so locate finds new files
```

### Tip 3: Use `plocate` (modern replacement)
```bash
$ sudo apt install plocate
# plocate is FASTER and uses less memory than mlocate
```

### Tip 4: Exclude directories from database
```bash
$ cat /etc/updatedb.conf
PRUNE_BIND_MOUNTS="yes"
PRUNEPATHS="/tmp /var/tmp /proc /sys /dev /run"
PRUNEFS="NFS nfs nfs4 rpc_pipefs afs binfmt_misc"
```

### Tip 5: Forensics — hidden files
```bash
# Find all hidden files on system
$ locate "/\." | grep -v "cache\|local\|config"
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Blazing fast (milliseconds) | Database can be outdated |
| Simple syntax | Cannot search by size/time/perms |
| Case-insensitive option | Cannot execute actions |
| Regex support | Needs `updatedb` (root) |
| Respects permissions | Only searches by name |

---

## 📍 Where & When to Use

| Scenario | Use `locate`? | Better Alternative |
|----------|-------------|-------------------|
| Quick filename search | ✅ Yes | — |
| Search by size/time/perms | ❌ No | `find` |
| Always need current results | ❌ No | `find` |
| Search inside files | ❌ No | `grep`, `find + grep` |
| Large filesystem search | ✅ Yes | — |
| Execute actions on results | ❌ No | `find -exec` |
| Fuzzy/interactive search | ❌ No | `fzf`, `fd` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| "command not found" | Install: `sudo apt install mlocate` |
| No results after install | Run `sudo updatedb` first |
| Results show deleted files | Use `locate -e` or run `updatedb` |
| Can't find new files | Run `sudo updatedb` |
| Seeing files without access | locate respects perms |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Install `locate` and update the database
2. Find all files named "bashrc"
3. Find all `.conf` files (case insensitive)

### 🟡 Intermediate
4. Count total `.py` files on system
5. Show database statistics
6. Limit results to first 10 matches
7. Find files only by basename

### 🔴 Advanced
8. Use regex to find `.py` or `.js` files
9. Create a custom database for your projects
10. Combine `locate` with `grep` to find config files containing specific text
11. Compare speed: `locate "*.md"` vs `find / -name "*.md"`

---

## 🧠 Cheat Sheet

```
sudo updatedb                  → Update database
locate filename                → Basic search
locate -i filename             → Case insensitive
locate -n 10 pattern           → Limit to 10 results
locate -c pattern              → Count matches
locate -b pattern              → Basename only
locate -e pattern              → Only existing files
locate -r "regex"              → Regex search
locate -S                      → Database stats

INSTALL: sudo apt install mlocate (or plocate)
UPDATE:  sudo updatedb (run after file changes)
CONFIG:  /etc/updatedb.conf (exclude paths)
```

---

> **Previous**: [`find` ←](./09_find.md) | **Next Category**: [📝 Text Processing →](../02_text_processing/)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
