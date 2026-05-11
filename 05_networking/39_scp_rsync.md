# 🛠️ `scp` & `rsync` — Secure File Copy & Sync | Linux Master Note

> **Transfer files between machines securely. `scp` is simple. `rsync` is powerful. Together they cover every file transfer scenario.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--scp-vs-rsync)
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

### `scp` vs `rsync`:
| Feature | `scp` | `rsync` |
|---------|-------|---------|
| Simplicity | ✅ Very simple | ⚠️ More options |
| Delta transfer | ❌ Copies everything | ✅ Only changed parts |
| Resume transfer | ❌ No | ✅ Yes (`--partial`) |
| Preserve permissions | Basic | ✅ Full (`-a`) |
| Bandwidth limiting | ❌ No (yes in newer) | ✅ `--bwlimit` |
| Exclude files | ❌ No | ✅ `--exclude` |
| Dry run | ❌ No | ✅ `--dry-run` |
| Progress | `-v` | `--progress` |

> 🎯 **Use `scp` for quick one-off copies. Use `rsync` for everything else.**

---

## 🧰 Syntax & Options

### `scp` syntax:
```bash
scp [OPTIONS] SOURCE DESTINATION
# Remote format: user@host:/path
```

| Flag | Description |
|------|-------------|
| `-r` | Recursive (directories) |
| `-P PORT` | SSH port (capital P!) |
| `-i KEY` | Identity file |
| `-p` | Preserve timestamps |
| `-C` | Enable compression |
| `-q` | Quiet mode |
| `-l KBITS` | Limit bandwidth |
| `-3` | Copy through local host |

### `rsync` syntax:
```bash
rsync [OPTIONS] SOURCE DESTINATION
```

| Flag | Description |
|------|-------------|
| `-a` | **Archive** mode (recursive + perms + times + symlinks) |
| `-v` | Verbose |
| `-z` | Compress during transfer |
| `-P` | Progress + partial (resume) |
| `--progress` | Show progress per file |
| `--partial` | Keep partial files (resume) |
| `-h` | Human-readable sizes |
| `-n` / `--dry-run` | Simulate (don't actually copy) |
| `--delete` | Delete files in dest not in source |
| `--exclude=PATTERN` | Exclude files matching pattern |
| `--include=PATTERN` | Include files matching pattern |
| `--bwlimit=KBPS` | Bandwidth limit |
| `-e "ssh -p PORT"` | Custom SSH command |
| `--backup` | Make backups of replaced files |
| `--backup-dir=DIR` | Directory for backups |
| `-u` / `--update` | Skip files newer in destination |
| `--chmod=PERMS` | Set permissions on destination |
| `--chown=USER:GROUP` | Set ownership on destination |
| `--max-size=SIZE` | Skip files larger than SIZE |

---

## 🟢 Basic Usage

### `scp` basics:
```bash
# Copy local file to remote
$ scp file.txt user@server:/home/user/
$ scp report.pdf dipro@192.168.1.100:~/Documents/

# Copy remote file to local
$ scp user@server:/var/log/app.log ./

# Copy directory (recursive)
$ scp -r project/ user@server:/opt/apps/

# Custom SSH port
$ scp -P 2222 file.txt user@server:/tmp/

# With specific key
$ scp -i ~/.ssh/my_key file.txt user@server:/tmp/

# Copy between two remote hosts
$ scp user1@server1:/path/file user2@server2:/path/
```

### `rsync` basics:
```bash
# Sync local to remote
$ rsync -avz project/ user@server:/opt/apps/project/

# Sync remote to local
$ rsync -avz user@server:/var/www/html/ ./backup/

# With progress
$ rsync -avzP large_file.iso user@server:/data/

# Dry run (see what would happen)
$ rsync -avzn project/ user@server:/opt/apps/project/
```

> ⚠️ **Trailing slash matters in rsync!**
> - `rsync source/ dest/` → copies CONTENTS of source into dest
> - `rsync source dest/` → copies source DIRECTORY into dest

---

## 🟡 Intermediate Usage

### Exclude files/directories
```bash
# Exclude patterns
$ rsync -avz --exclude='node_modules' --exclude='.git' --exclude='*.log' \
  project/ user@server:/opt/apps/project/

# Exclude from file
$ cat exclude.txt
node_modules
.git
*.log
*.pyc
__pycache__

$ rsync -avz --exclude-from=exclude.txt project/ user@server:/opt/apps/
```

### Delete files not in source (mirror)
```bash
# Mirror exactly — delete extra files on destination
$ rsync -avz --delete project/ user@server:/opt/apps/project/

# Dry run first to see what would be deleted!
$ rsync -avzn --delete project/ user@server:/opt/apps/project/
```

### Bandwidth limiting
```bash
# Limit to 1MB/s
$ rsync -avzP --bwlimit=1000 large_dir/ user@server:/backup/
```

### Custom SSH options
```bash
# Use different port
$ rsync -avz -e "ssh -p 2222" project/ user@server:/opt/

# Use specific key
$ rsync -avz -e "ssh -i ~/.ssh/deploy_key" project/ user@server:/opt/
```

### Backup with versioning
```bash
# Keep backups of replaced files
$ rsync -avz --backup --backup-dir=/backup/$(date +%Y%m%d) \
  /var/www/html/ /backup/current/
```

---

## 🔴 Advanced Usage

### Automated Backup Script
```bash
#!/bin/bash
# backup_rsync.sh — Daily incremental backup
SOURCE="/var/www/html/"
DEST="backup@backup-server:/backups/web/"
LOG="/var/log/backup.log"
EXCLUDE="--exclude=cache --exclude=tmp --exclude='*.log'"

echo "$(date): Starting backup..." >> "$LOG"

rsync -avz --delete --partial $EXCLUDE \
  -e "ssh -i /root/.ssh/backup_key" \
  "$SOURCE" "$DEST" >> "$LOG" 2>&1

if [ $? -eq 0 ]; then
    echo "$(date): Backup SUCCESS" >> "$LOG"
else
    echo "$(date): Backup FAILED!" >> "$LOG"
fi
```

### Deploy web application
```bash
#!/bin/bash
# deploy.sh — Deploy to production
APP_DIR="/home/dipro/myapp/"
SERVER="deploy@prod.example.com"
REMOTE_DIR="/var/www/myapp/"

echo "Deploying to production..."

# Sync files (exclude dev stuff)
rsync -avz --delete \
  --exclude='.git' \
  --exclude='node_modules' \
  --exclude='.env.local' \
  --exclude='tests/' \
  -e "ssh -i ~/.ssh/deploy_key" \
  "$APP_DIR" "$SERVER:$REMOTE_DIR"

# Restart service
ssh -i ~/.ssh/deploy_key "$SERVER" "sudo systemctl restart myapp"

echo "Deployment complete!"
```

### CTF/Pentesting — File Exfiltration 🎯
```bash
# Quickly grab interesting files from compromised host
$ scp -i found_key user@target:/etc/shadow ./loot/
$ scp -i found_key user@target:/etc/passwd ./loot/
$ scp -r -i found_key user@target:/home/ ./loot/home/

# rsync specific file types
$ rsync -avz --include='*.conf' --include='*.key' --include='*.pem' \
  --exclude='*' user@target:/etc/ ./loot/configs/

# Transfer tools TO target
$ scp linpeas.sh user@target:/tmp/
$ scp -i key chisel user@target:/tmp/
```

### Sync with checksums (verify integrity)
```bash
# Use checksums instead of timestamps
$ rsync -avzc source/ dest/
# -c = compare by checksum, not timestamp/size
```

---

## 💡 Real World Pro Tips

### Tip 1: rsync `-avzP` is your go-to combo
```bash
$ rsync -avzP source/ dest/
# -a = archive, -v = verbose, -z = compress, -P = progress+partial
```

### Tip 2: Always dry-run before `--delete`
```bash
# See what would be deleted FIRST
$ rsync -avzn --delete source/ dest/
# If OK, run without -n
$ rsync -avz --delete source/ dest/
```

### Tip 3: Trailing slash matters!
```bash
# WITH slash: copies contents
$ rsync -avz project/ server:/opt/project/
# Result: /opt/project/file.txt

# WITHOUT slash: copies directory
$ rsync -avz project server:/opt/
# Result: /opt/project/file.txt
```

### Tip 4: `scp` is being deprecated
```bash
# OpenSSH 9.0+ recommends sftp/rsync over scp
# scp has security issues with remote-to-remote copies
# Use rsync or sftp instead for new projects
```

---

## ✅ Pros & Cons

### `scp`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Dead simple syntax | No delta/incremental |
| Pre-installed everywhere | No resume |
| Encrypted (SSH) | Being deprecated |

### `rsync`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Delta transfers (fast!) | More complex syntax |
| Resume support | Trailing slash confusion |
| Exclude/include patterns | May need installation |
| Dry run mode | `--delete` can be dangerous |
| Bandwidth limiting | — |

---

## 📍 Where & When to Use

| Scenario | Tool | Why |
|----------|------|-----|
| Quick single file copy | `scp` | Simple |
| Large directory sync | `rsync` | Delta transfer |
| Daily backups | `rsync` | Incremental |
| Web deployment | `rsync` | Exclude dev files |
| Resume large transfer | `rsync -P` | Partial support |
| CTF file grab | `scp` | Fast, simple |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| `rsync` trailing slash confusion | `source/` = contents, `source` = directory |
| `rsync --delete` without dry-run | Always `-n` first! |
| `scp -p 2222` (lowercase p) | Use `-P 2222` (uppercase!) |
| Not preserving permissions | Use `rsync -a` |
| Slow for unchanged files | Use `rsync` instead of `scp` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Copy a file to a remote server with `scp`
2. Copy a directory with `scp -r`
3. Sync a directory with `rsync -avz`

### 🟡 Intermediate
4. Use `rsync` with exclude patterns
5. Do a dry-run before syncing with `--delete`
6. Resume an interrupted transfer with `rsync -P`

### 🔴 Advanced
7. Write an automated backup script with rsync
8. Create a deployment script for a web app
9. Set up incremental backups with backup-dir

---

## 🧠 Cheat Sheet

```
SCP:
  scp file user@host:/path         → Upload file
  scp user@host:/path/file ./      → Download file
  scp -r dir/ user@host:/path/     → Upload directory
  scp -P 2222 file user@host:/     → Custom port
  scp -i key file user@host:/      → Custom key

RSYNC:
  rsync -avzP source/ user@host:/dest/     → Sync with progress
  rsync -avz --delete src/ dest/           → Mirror (delete extra)
  rsync -avzn src/ dest/                   → Dry run
  rsync -avz --exclude='.git' src/ dest/   → With exclusions
  rsync -avz --bwlimit=1000 src/ dest/     → Bandwidth limit
  rsync -avz -e "ssh -p 2222" src/ dest/   → Custom SSH port

REMEMBER:
  source/  → copy CONTENTS
  source   → copy DIRECTORY
  Always --dry-run before --delete!
```

---

> **Previous**: [`ssh` ←](./38_ssh.md) | **Next**: [`nmap` →](./40_nmap.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
