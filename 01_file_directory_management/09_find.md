# 🛠️ `find` — Search Files in Directory Hierarchy | Linux Master Note

> **The Swiss Army knife of file searching. `find` is one of the most POWERFUL Linux commands ever created.**

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

### What is `find`?
`find` searches for files and directories in a directory hierarchy based on various criteria (name, size, time, permissions, type) and can execute actions on the results.

### Why is it powerful?
- Searches by name, size, time, type, permissions, owner, and MORE
- Can execute commands on found files
- Supports complex logical expressions (AND, OR, NOT)
- Real-time search (not indexed like `locate`)

---

## 📖 Theory

### How `find` works internally:
1. Starts at the given directory
2. Recursively traverses the directory tree using `readdir()`
3. Calls `stat()` on each file to get metadata
4. Tests each file against the given criteria
5. Performs the specified action on matching files

### Difference from `locate`:
- `find`: **Real-time** search — always up-to-date, but slower
- `locate`: **Database** search — fast but may be outdated

---

## 🧰 Syntax & Options

```bash
find [PATH] [OPTIONS] [EXPRESSION]
```

### Search Criteria

| Test | Description | Example |
|------|-------------|---------|
| `-name` | Match filename (case sensitive) | `-name "*.txt"` |
| `-iname` | Match filename (case insensitive) | `-iname "*.TXT"` |
| `-type` | File type: `f`(file), `d`(dir), `l`(link), `s`(socket) | `-type f` |
| `-size` | File size: `+`(larger), `-`(smaller), `c`(bytes), `k`(KB), `M`(MB), `G`(GB) | `-size +100M` |
| `-mtime` | Modified time in days: `+n`(older), `-n`(newer) | `-mtime -7` |
| `-atime` | Access time in days | `-atime +30` |
| `-ctime` | Change time in days | `-ctime -1` |
| `-mmin` | Modified time in minutes | `-mmin -60` |
| `-perm` | Permission match | `-perm 755` |
| `-user` | Owner username | `-user dipro` |
| `-group` | Group name | `-group admin` |
| `-empty` | Empty files or directories | `-empty` |
| `-maxdepth` | Maximum search depth | `-maxdepth 2` |
| `-mindepth` | Minimum search depth | `-mindepth 1` |
| `-newer` | Newer than reference file | `-newer ref.txt` |
| `-regex` | Match full path with regex | `-regex ".*\.py$"` |

### Actions

| Action | Description | Example |
|--------|-------------|---------|
| `-print` | Print pathname (default) | `-print` |
| `-print0` | Print null-terminated (for xargs) | `-print0` |
| `-delete` | Delete found files | `-delete` |
| `-exec` | Execute command on each file | `-exec cmd {} \;` |
| `-exec +` | Execute command on all files at once | `-exec cmd {} +` |
| `-ok` | Like `-exec` but asks confirmation | `-ok rm {} \;` |
| `-ls` | Long listing of found files | `-ls` |

### Logical Operators

| Operator | Description |
|----------|-------------|
| `-and` / `-a` | AND (default) |
| `-or` / `-o` | OR |
| `-not` / `!` | NOT |
| `( )` | Grouping (escape: `\( \)`) |

---

## 🟢 Basic Usage

### Find by name
```bash
# Find file named "config.txt"
$ find / -name "config.txt"

# Case-insensitive search
$ find /home -iname "readme.md"

# Find all .log files
$ find /var/log -name "*.log"
```

### Find by type
```bash
# Find only files
$ find /etc -type f

# Find only directories
$ find /home -type d

# Find only symbolic links
$ find /usr -type l
```

### Find by size
```bash
# Files larger than 100MB
$ find / -size +100M

# Files smaller than 1KB
$ find . -size -1k

# Files exactly 0 bytes (empty)
$ find . -empty
```

---

## 🟡 Intermediate Usage

### Find by time
```bash
# Modified in last 7 days
$ find /home -mtime -7

# Modified MORE than 30 days ago
$ find /var/log -mtime +30

# Accessed in last 60 minutes
$ find . -amin -60

# Modified in last 2 hours
$ find . -mmin -120
```

### Find by permissions
```bash
# Files with exact permission 777
$ find / -perm 777

# Files with at least SUID bit set
$ find / -perm -4000

# Files writable by others
$ find / -perm -o=w

# Files NOT readable by group
$ find . ! -perm -g=r
```

### Find by owner
```bash
# Files owned by user
$ find /home -user dipro

# Files owned by root
$ find /etc -user root -type f

# Files with no owner (orphaned)
$ find / -nouser
```

### Limit search depth
```bash
# Only current directory (no recursion)
$ find . -maxdepth 1 -name "*.txt"

# Only 2 levels deep
$ find /etc -maxdepth 2 -type f

# Skip current directory level
$ find . -mindepth 1 -type d
```

---

## 🔴 Advanced Usage

### Execute commands on found files
```bash
# Delete all .tmp files
$ find /tmp -name "*.tmp" -delete

# Remove old log files
$ find /var/log -name "*.log" -mtime +90 -exec rm {} \;

# Change permissions of all .sh files
$ find . -name "*.sh" -exec chmod +x {} \;

# Copy found files to backup
$ find . -name "*.conf" -exec cp {} /backup/ \;

# More efficient — pass multiple files at once
$ find . -name "*.txt" -exec wc -l {} +
```

### Complex logical expressions
```bash
# Files that are .py OR .js
$ find . \( -name "*.py" -o -name "*.js" \) -type f

# Files larger than 10MB AND modified in last week
$ find /home -size +10M -and -mtime -7

# Files NOT owned by root
$ find /etc ! -user root

# Complex: (.log files older than 30 days) OR (empty files)
$ find . \( -name "*.log" -mtime +30 \) -o \( -empty -type f \)
```

### Security auditing with find 🔒
```bash
# Find SUID files (privilege escalation risk!)
$ find / -perm -4000 -type f 2>/dev/null

# Find SGID files
$ find / -perm -2000 -type f 2>/dev/null

# Find world-writable files
$ find / -perm -o=w -type f 2>/dev/null

# Find files with no owner (potential backdoor)
$ find / -nouser -o -nogroup 2>/dev/null

# Find files modified in last 24 hours (incident response)
$ find / -mtime -1 -type f 2>/dev/null

# Find executable files in /tmp (suspicious!)
$ find /tmp -type f -executable 2>/dev/null

# Find hidden files (start with .)
$ find / -name ".*" -type f 2>/dev/null
```

### Using `-print0` with `xargs -0` (safe for special filenames)
```bash
# Safely handle filenames with spaces/newlines
$ find . -name "*.txt" -print0 | xargs -0 wc -l

# Safe deletion
$ find /tmp -name "*.tmp" -print0 | xargs -0 rm -v
```

### Find with regex
```bash
# Files matching regex pattern
$ find . -regex ".*\.\(py\|js\|ts\)$"

# Using extended regex
$ find . -regextype posix-extended -regex ".*\.(py|js|ts)$"
```

---

## 🔗 Piping & Combining

```bash
# find + grep — Search inside found files
$ find . -name "*.py" -exec grep -l "import os" {} +

# find + wc — Count lines in all Python files
$ find . -name "*.py" -exec cat {} + | wc -l

# find + tar — Archive found files
$ find . -name "*.log" -mtime +30 | tar czf old_logs.tar.gz -T -

# find + du — Size of found items
$ find . -name "*.mp4" -exec du -sh {} +

# find + sed — Replace text in multiple files
$ find . -name "*.txt" -exec sed -i 's/old/new/g' {} +
```

---

## 💡 Real World Pro Tips

### Tip 1: Suppress permission errors
```bash
$ find / -name "secret.txt" 2>/dev/null
```

### Tip 2: Find duplicate files (by size)
```bash
$ find . -type f -exec md5sum {} + | sort | uniq -d -w 32
```

### Tip 3: Disk space cleanup
```bash
# Find top 10 largest files
$ find / -type f -exec du -h {} + 2>/dev/null | sort -rh | head -10
```

### Tip 4: Combine with `stat` for detailed info
```bash
$ find . -name "*.conf" -exec stat --printf="%n %s %U %A\n" {} +
```

### Tip 5: Watch for real-time file changes
```bash
# Find files modified in last 5 minutes (use in cron)
$ find /var/www -mmin -5 -type f
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Extremely powerful & flexible | Slow on large filesystems |
| Real-time results (always accurate) | Complex syntax |
| Can execute actions on results | No indexing (searches every time) |
| Supports logical expressions | `find / ...` scans entire disk |
| Security auditing capability | Permission errors without `2>/dev/null` |

---

## 📍 Where & When to Use

| Scenario | Use `find`? | Alternative |
|----------|-----------|-------------|
| Search by name | ✅ Yes | `locate` (faster) |
| Search by size/time/perms | ✅ Yes | — |
| Security auditing | ✅ Yes | `lynis` |
| Batch file operations | ✅ `-exec` | `xargs` |
| Quick filename search | ❌ Slow | `locate`, `fd` |
| Real-time monitoring | ⚠️ With cron | `inotifywait` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Not quoting patterns | Use `"*.txt"` not `*.txt` |
| Forgetting `2>/dev/null` | Always add for system-wide search |
| Using `\;` instead of `+` | `+` is faster (batch mode) |
| `-delete` without testing | Run without `-delete` first! |
| Wrong `-mtime` math | `-mtime -1` = last 24hrs, not today |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Find all `.txt` files in your home directory
2. Find all directories in `/etc`
3. Find all empty files in `/tmp`

### 🟡 Intermediate
4. Find files larger than 50MB on the system
5. Find files modified in the last 3 days
6. Find all SUID files on the system
7. Find and delete all `.tmp` files in `/tmp`

### 🔴 Advanced
8. Find all `.py` files containing "import os" 
9. Find world-writable files (security audit)
10. Write a cleanup script that archives logs older than 30 days
11. Find files modified in last hour with specific permissions

---

## 🧠 Cheat Sheet

```
find . -name "*.txt"           → By name
find . -type f/d/l             → By type (file/dir/link)
find . -size +100M             → By size
find . -mtime -7               → Modified last 7 days
find . -perm 755               → By permissions
find . -user root              → By owner
find . -empty                  → Empty files
find . -maxdepth 2             → Limit depth
find . -exec cmd {} \;         → Execute (one by one)
find . -exec cmd {} +          → Execute (batch)
find . -delete                 → Delete found files
find . \( -name "*.py" -o -name "*.js" \)  → OR logic
find . -name "*.tmp" -print0 | xargs -0 rm → Safe pipe

SECURITY: find / -perm -4000   → SUID files
          find / -perm -o=w    → World-writable
          find / -nouser       → Orphaned files
```

---

> **Previous**: [`touch` ←](./08_touch.md) | **Next**: [`locate` →](./10_locate.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
