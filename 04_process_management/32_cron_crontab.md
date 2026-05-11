# 🛠️ `cron` & `crontab` — Schedule Recurring Tasks | Linux Master Note

> **The Linux task scheduler. Automate backups, log rotation, security scans, updates — anything that needs to run on a schedule.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--cron-architecture)
3. [Syntax & Options](#-syntax--crontab-format)
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

### What is `cron`?
`cron` is a **time-based job scheduler** daemon that runs in the background. `crontab` (**cron tab**le) is the command to manage scheduled tasks.

### The cron family:
| Component | Purpose |
|-----------|---------|
| `crond` | The background daemon that executes jobs |
| `crontab` | CLI tool to edit/view/manage cron jobs |
| `/etc/crontab` | System-wide cron table |
| `/etc/cron.d/` | Drop-in cron job files |
| `/etc/cron.{hourly,daily,weekly,monthly}/` | Pre-configured schedule directories |

### Why it matters:
- **Backups** — Automatic daily/weekly backups
- **Log rotation** — Clean up old logs
- **Monitoring** — Health checks every 5 minutes
- **Security** — Scheduled vulnerability scans
- **Updates** — Automated package updates

---

## 📖 Theory — Cron Architecture

### How cron works:
```
1. crond daemon starts at boot
2. Every minute, it wakes up
3. Reads all crontab files
4. Checks if any job matches current time
5. Executes matching jobs as the specified user
6. Goes back to sleep for 1 minute
```

### Cron files locations:
| Location | Type | Purpose |
|----------|------|---------|
| `/var/spool/cron/crontabs/USERNAME` | User crontabs | User's personal jobs |
| `/etc/crontab` | System crontab | System-wide jobs (has USER field) |
| `/etc/cron.d/*` | Drop-in files | Package-specific jobs |
| `/etc/cron.hourly/` | Scripts | Run every hour |
| `/etc/cron.daily/` | Scripts | Run every day |
| `/etc/cron.weekly/` | Scripts | Run every week |
| `/etc/cron.monthly/` | Scripts | Run every month |

### Cron environment:
> ⚠️ Cron runs with a **minimal environment** — NOT your normal shell!
```
PATH=/usr/bin:/bin
HOME=/home/username
SHELL=/bin/sh
# No aliases, no .bashrc, no custom PATH!
```

---

## 🧰 Syntax — Crontab Format

### The 5-field time specification:
```
┌───────── minute (0 - 59)
│ ┌───────── hour (0 - 23)
│ │ ┌───────── day of month (1 - 31)
│ │ │ ┌───────── month (1 - 12)
│ │ │ │ ┌───────── day of week (0 - 7, 0 and 7 = Sunday)
│ │ │ │ │
* * * * * command_to_execute
```

### Special characters:
| Char | Meaning | Example |
|------|---------|---------|
| `*` | Every value | `* * * * *` = every minute |
| `,` | List of values | `1,15,30` = at 1, 15, 30 |
| `-` | Range | `9-17` = 9 through 17 |
| `/` | Step/interval | `*/5` = every 5 units |
| `@` | Shortcut | `@daily` = once a day |

### Schedule shortcuts:
| Shortcut | Equivalent | Meaning |
|----------|-----------|---------|
| `@reboot` | — | Run once at startup |
| `@yearly` / `@annually` | `0 0 1 1 *` | Jan 1st, midnight |
| `@monthly` | `0 0 1 * *` | 1st of month, midnight |
| `@weekly` | `0 0 * * 0` | Sunday, midnight |
| `@daily` / `@midnight` | `0 0 * * *` | Every day, midnight |
| `@hourly` | `0 * * * *` | Every hour |

### `crontab` command options:
| Flag | Description |
|------|-------------|
| `crontab -e` | Edit your crontab |
| `crontab -l` | List your crontab |
| `crontab -r` | Remove your crontab (⚠️ deletes ALL!) |
| `crontab -u USER -e` | Edit another user's crontab (root) |
| `crontab -u USER -l` | List another user's crontab (root) |

---

## 🟢 Basic Usage

### Edit crontab
```bash
$ crontab -e     # Opens editor (usually nano or vi)
```

### Common schedule examples:
```bash
# Every minute
* * * * * /path/to/script.sh

# Every 5 minutes
*/5 * * * * /path/to/script.sh

# Every hour (at minute 0)
0 * * * * /path/to/script.sh

# Every day at 2:30 AM
30 2 * * * /path/to/script.sh

# Every Monday at 9 AM
0 9 * * 1 /path/to/script.sh

# Every 1st of month at midnight
0 0 1 * * /path/to/script.sh

# At system boot
@reboot /path/to/startup_script.sh
```

### List and verify
```bash
$ crontab -l
*/5 * * * * /home/dipro/scripts/health_check.sh
0 2 * * * /home/dipro/scripts/backup.sh
```

---

## 🟡 Intermediate Usage

### Output handling
```bash
# Redirect output to log file
*/5 * * * * /path/to/script.sh >> /var/log/myjob.log 2>&1

# Discard all output (silent)
*/5 * * * * /path/to/script.sh > /dev/null 2>&1

# Email output (default: sends to user's mail)
MAILTO="dipro@example.com"
*/5 * * * * /path/to/script.sh

# Disable email
MAILTO=""
*/5 * * * * /path/to/script.sh
```

### Environment variables in crontab
```bash
# Set PATH (crucial — cron has minimal PATH!)
PATH=/usr/local/bin:/usr/bin:/bin:/home/dipro/.local/bin
SHELL=/bin/bash
HOME=/home/dipro
MAILTO=""

# Now your scripts can find commands
*/5 * * * * python3 /home/dipro/app/check.py
```

### Multiple schedules
```bash
# Business hours only (9 AM - 5 PM, Mon-Fri)
*/15 9-17 * * 1-5 /path/to/work_task.sh

# Weekdays at 6 AM and 6 PM
0 6,18 * * 1-5 /path/to/report.sh

# Every 2 hours
0 */2 * * * /path/to/task.sh

# First Monday of every month (trick!)
0 9 1-7 * 1 /path/to/monthly_report.sh
```

### System crontab (`/etc/crontab`)
```bash
# /etc/crontab has an extra USER field
# min hour dom mon dow USER command
*/5 * * * * root /usr/local/bin/health_check.sh
0   2 * * * backup /opt/backup/run_backup.sh
```

### Drop-in cron files
```bash
# Create /etc/cron.d/myapp
$ sudo cat > /etc/cron.d/myapp << 'EOF'
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
*/5 * * * * www-data /opt/myapp/cleanup.sh
EOF
```

---

## 🔴 Advanced Usage

### Automated Backup System
```bash
# crontab -e
# Daily full backup at 2 AM
0 2 * * * /home/dipro/scripts/backup.sh >> /var/log/backup.log 2>&1

# /home/dipro/scripts/backup.sh
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup"
SOURCE="/var/www/html"

# Create backup
tar -czf "$BACKUP_DIR/web_$DATE.tar.gz" "$SOURCE"

# Database backup
mysqldump -u root --all-databases > "$BACKUP_DIR/db_$DATE.sql"

# Remove backups older than 30 days
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete
find "$BACKUP_DIR" -name "*.sql" -mtime +30 -delete

echo "$(date): Backup completed successfully"
```

### Security Monitoring Cron 🔒
```bash
# Check for new SUID files every hour
0 * * * * find / -perm -4000 -type f 2>/dev/null | sort > /tmp/suid_current.txt && diff /var/lib/suid_baseline.txt /tmp/suid_current.txt | mail -s "SUID Alert" admin@example.com

# Check for rootkits daily
0 3 * * * /usr/bin/chkrootkit 2>&1 | mail -s "Rootkit Check" admin@example.com

# Monitor failed SSH logins
*/30 * * * * grep "Failed password" /var/log/auth.log | tail -20 >> /var/log/ssh_failures.log

# Check for unauthorized users
0 */6 * * * awk -F: '$3 == 0 && $1 != "root" {print "WARNING: UID 0 user:", $1}' /etc/passwd | mail -s "UID 0 Alert" admin@example.com
```

### Preventing Overlap (Flock)
```bash
# Problem: What if cron job takes longer than interval?
# Solution: Use flock to prevent overlapping

*/5 * * * * /usr/bin/flock -n /tmp/myjob.lock /path/to/script.sh

# -n = non-blocking (skip if locked)
# If previous run is still going, this one is skipped
```

### CTF/Pentesting — Cron Exploitation 🎯
```bash
# Enumerate cron jobs (privilege escalation recon!)
$ cat /etc/crontab
$ ls -la /etc/cron.*
$ crontab -l
$ cat /var/spool/cron/crontabs/*  2>/dev/null

# Find writable cron scripts (privesc vector!)
$ find /etc/cron* -writable 2>/dev/null
$ ls -la /etc/cron.d/

# Cron PATH hijacking:
# If cron runs "backup.sh" without full path, and cron PATH
# includes a writable directory, you can hijack it!

# Monitor cron execution (see what runs)
# Tool: pspy (https://github.com/DominicBreuker/pspy)
$ ./pspy64     # Watch all processes including cron jobs

# Wildcard injection in cron scripts:
# If cron runs: tar czf backup.tar.gz *
# Create files named: --checkpoint=1 --checkpoint-action=exec=shell.sh
```

---

## 🔗 Piping & Combining

```bash
# List all cron jobs for all users (root only)
$ for user in $(cut -d: -f1 /etc/passwd); do
    echo "=== $user ==="
    sudo crontab -u "$user" -l 2>/dev/null
done

# Monitor cron execution in real-time
$ sudo tail -f /var/log/syslog | grep CRON
$ sudo journalctl -u cron -f

# Backup all crontabs
$ sudo tar czf crontabs_backup.tar.gz /var/spool/cron/ /etc/crontab /etc/cron.d/
```

---

## 💡 Real World Pro Tips

### Tip 1: Always use full paths!
```bash
# ❌ Cron can't find 'python3'
*/5 * * * * python3 /app/script.py

# ✅ Use full path
*/5 * * * * /usr/bin/python3 /app/script.py

# Find full path:
$ which python3
/usr/bin/python3
```

### Tip 2: Test your cron command manually first!
```bash
# Run EXACTLY what cron will run:
$ /bin/bash -c '/usr/bin/python3 /home/dipro/script.py'
# If this works → cron will work
```

### Tip 3: Debug cron failures
```bash
# Check cron logs
$ sudo grep CRON /var/log/syslog | tail -20
$ sudo journalctl -u cron --since "1 hour ago"

# Add logging to your cron command
*/5 * * * * /path/to/script.sh >> /tmp/cron_debug.log 2>&1
```

### Tip 4: Use `crontab.guru` to build schedules
```
https://crontab.guru
# Interactive cron schedule builder — bookmark it!
```

### Tip 5: Backup before editing
```bash
$ crontab -l > ~/crontab_backup_$(date +%Y%m%d).txt
$ crontab -e
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Built-in on every Linux | Minimal environment (PATH issues) |
| Minute-level precision | No dependency management |
| Per-user crontabs | No retry on failure |
| System-wide cron.d | Hard to debug |
| @reboot for startup | No overlap prevention (need flock) |
| Logs to syslog | Confusing day-of-week numbering |

---

## 📍 Where & When to Use

| Scenario | Schedule | Example |
|----------|----------|---------|
| Backup | Daily 2 AM | `0 2 * * *` |
| Log rotation | Weekly Sunday | `0 0 * * 0` |
| Health check | Every 5 min | `*/5 * * * *` |
| Security scan | Daily 3 AM | `0 3 * * *` |
| Report generation | Mon-Fri 9 AM | `0 9 * * 1-5` |
| Startup task | At boot | `@reboot` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Not using full paths | Use `which cmd` and use absolute path |
| Forgetting output redirect | Add `>> log 2>&1` |
| Editing with `crontab -r` (delete!) | Use `crontab -e` to edit |
| Cron has different PATH | Set PATH in crontab |
| Job overlaps with itself | Use `flock` for locking |
| Not testing command first | Run manually with `/bin/bash -c` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Create a cron job that logs the date every minute
2. List all your cron jobs with `crontab -l`
3. Create a job that runs at boot with `@reboot`

### 🟡 Intermediate
4. Set up a daily backup script at 2 AM
5. Create a cron job that runs only on weekdays
6. Redirect cron output to a log file

### 🔴 Advanced
7. Set up a security monitoring cron with email alerts
8. Use flock to prevent job overlap
9. Enumerate all system cron jobs for security audit

---

## 🧠 Cheat Sheet

```
FORMAT:  min hour dom mon dow command
         *   *    *   *   *

EXAMPLES:
  * * * * *          → Every minute
  */5 * * * *        → Every 5 minutes
  0 * * * *          → Every hour
  0 2 * * *          → Daily at 2 AM
  0 9 * * 1-5        → Weekdays at 9 AM
  0 0 * * 0          → Weekly (Sunday midnight)
  0 0 1 * *          → Monthly (1st, midnight)
  @reboot            → At system startup

COMMANDS:
  crontab -e         → Edit crontab
  crontab -l         → List crontab
  crontab -r         → DELETE crontab (careful!)

OUTPUT:
  cmd >> log 2>&1    → Log output
  cmd > /dev/null 2>&1  → Silence
  MAILTO=""          → No email

DEBUG:
  grep CRON /var/log/syslog    → Check logs
  which python3                → Get full path
  flock -n /tmp/j.lock cmd     → Prevent overlap

AUDIT:
  cat /etc/crontab             → System crontab
  ls /etc/cron.d/              → Drop-in jobs
  sudo crontab -u user -l     → User's crontab
```

---

> **Previous**: [`nohup` ←](./31_nohup.md) | **Next**: [`ping` →](../05_networking/33_ping.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
