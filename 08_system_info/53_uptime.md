# ⏱️ `uptime` — System Uptime & Load Average | Linux Master Note

> **How long has the system been running? How busy is it right now? `uptime` answers both in one line — a critical pulse-check for sysadmins, pentesters, and anyone monitoring Linux systems.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory — Uptime & Load Averages](#-theory--uptime--load-averages)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [Security & Pentesting](#-security--pentesting)
8. [Real World Pro Tips](#-real-world-pro-tips)
9. [Pros & Cons](#-pros--cons)
10. [Where & When to Use](#-where--when-to-use)
11. [Common Mistakes](#-common-mistakes)
12. [Practice Exercises](#-practice-exercises)
13. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### What is `uptime`?
`uptime` displays a single, information-packed line showing:
- **Current time** — system clock
- **How long** the system has been running
- **Number of users** currently logged in
- **Load averages** for the last 1, 5, and 15 minutes

```
 14:23:07 up 42 days,  3:17,  2 users,  load average: 0.52, 0.38, 0.31
 ────────  ──────────────────  ───────  ──────────────────────────────────
 current   system uptime       logged   load averages (1m, 5m, 15m)
  time                         users
```

### Why it matters:
- **Server Health** — Is the system overloaded or running smoothly?
- **Monitoring** — Detect CPU spikes and resource exhaustion
- **Security Auditing** — Long uptime = potentially missing kernel patches
- **Incident Response** — When was the last reboot? Was it expected?
- **CTF/Pentesting** — Recon: uptime reveals patch habits and stability
- **Capacity Planning** — Load trends guide scaling decisions

---

## 📖 Theory — Uptime & Load Averages

### What `uptime` reveals at a glance:

| Component | Example | Description |
|-----------|---------|-------------|
| Current time | `14:23:07` | System clock (local timezone) |
| Uptime duration | `up 42 days, 3:17` | Time since last boot |
| User count | `2 users` | Logged-in sessions (tty + pts) |
| Load avg (1 min) | `0.52` | Average runnable + waiting processes (last 60s) |
| Load avg (5 min) | `0.38` | Average over last 5 minutes |
| Load avg (15 min) | `0.31` | Average over last 15 minutes |

### 🧠 Load Average Deep Dive

Load average is **the most misunderstood metric in Linux**. It does NOT mean CPU percentage.

#### What load average actually measures:
> The average number of processes that are either **running on the CPU** or **waiting for CPU time** (in the run queue), plus processes in **uninterruptible sleep** (usually waiting for disk I/O).

#### The traffic analogy:
| Load Average | Meaning (Single CPU) | Analogy |
|-------------|----------------------|---------|
| `0.00` | Completely idle | Empty highway |
| `0.50` | 50% utilized | Light traffic |
| `1.00` | Fully utilized | Highway at capacity |
| `2.00` | Overloaded — 1 process waiting | Traffic jam, cars queuing |
| `4.00` | Severely overloaded | Gridlock |

#### ⚠️ Critical: Load Average Is Per-System, Not Per-Core

```
Load Average = 4.00

On a 1-core system → 4x overloaded ❌ (3 processes waiting)
On a 4-core system → Perfectly balanced ✅ (1 process per core)
On a 8-core system → Only 50% utilized 🟢 (plenty of headroom)
```

**Golden Rule:**
```
If Load Average ≤ Number of CPU Cores → System is healthy ✅
If Load Average > Number of CPU Cores → System is overloaded ⚠️
```

#### Reading the three values (1 / 5 / 15 min):

| Pattern | 1 min | 5 min | 15 min | Interpretation |
|---------|-------|-------|--------|---------------|
| Spike | `8.50` | `2.10` | `1.05` | Sudden burst, likely resolving |
| Rising | `3.20` | `2.10` | `1.05` | Load increasing — investigate! |
| Falling | `1.05` | `2.10` | `3.20` | Recovery from previous spike |
| Sustained | `4.00` | `4.10` | `3.95` | Consistently high — capacity issue |
| Healthy | `0.40` | `0.35` | `0.38` | System running smoothly |

#### Understanding the numbers:
```
load average: 0.52, 0.38, 0.31
              ────  ────  ────
               │     │     └── 15-min average (long-term trend)
               │     └──────── 5-min average (medium-term trend)
               └────────────── 1-min average (current situation)
```

### Where does uptime get its data?

| Source | What It Contains |
|--------|-----------------|
| `/proc/uptime` | Raw uptime in seconds + idle time in seconds |
| `/proc/loadavg` | Load averages + running/total processes + last PID |
| `utmp` / `wtmp` | User session data + reboot records |
| Kernel scheduler | Real-time process queue metrics |

---

## 🧰 Syntax & Options

```bash
uptime [OPTIONS]
```

| Flag | Description | Example Output |
|------|-------------|----------------|
| *(none)* | Full default output | `14:23:07 up 42 days, 3:17, 2 users, load average: 0.52, 0.38, 0.31` |
| `-p` | **Pretty** format (human-readable uptime only) | `up 6 weeks, 0 days, 3 hours, 17 minutes` |
| `-s` | **Since** — boot date/time | `2026-04-13 11:06:22` |
| `-V` | Version info | `uptime from procps-ng 3.3.17` |
| `-h` | Help message | Usage summary |

### Related files:
| File / Path | Description |
|-------------|-------------|
| `/proc/uptime` | Raw uptime + idle time (seconds) |
| `/proc/loadavg` | Load averages + process counts |
| `/var/run/utmp` | Current login sessions |
| `/var/log/wtmp` | Historical login/reboot records |

---

## 🟢 Basic Usage

### Standard uptime output
```bash
# Default — the one-liner you'll use daily
$ uptime
 14:23:07 up 42 days,  3:17,  2 users,  load average: 0.52, 0.38, 0.31

# Human-readable uptime duration
$ uptime -p
up 6 weeks, 0 days, 3 hours, 17 minutes

# When was the system booted?
$ uptime -s
2026-04-13 11:06:22
```

### Quick health checks
```bash
# How many users are logged in?
$ uptime | grep -oP '\d+ user'
2 user

# Just the load averages
$ uptime | grep -oP 'load average: .*'
load average: 0.52, 0.38, 0.31

# Check CPU core count for context
$ nproc
4
```

### Reading `/proc/uptime` raw data
```bash
$ cat /proc/uptime
3632257.42 14012853.68
#  ──────────  ──────────
#  │            └── Total idle time in seconds (across ALL cores)
#  └─────────────── System uptime in seconds

# Convert to human-readable
$ awk '{printf "Uptime: %d days, %d hours, %d minutes\n",
        $1/86400, ($1%86400)/3600, ($1%3600)/60}' /proc/uptime
Uptime: 42 days, 3 hours, 17 minutes
```

### Reading `/proc/loadavg`
```bash
$ cat /proc/loadavg
0.52 0.38 0.31 2/487 19832
# ──── ──── ──── ─────  ─────
#  1m   5m  15m  │       └── Last PID assigned
#                └────────── Running/Total processes (2 running of 487)
```

---

## 🟡 Intermediate Usage

### The `w` Command — `who` + `uptime` Combined
```bash
# w = uptime header + detailed user list
$ w
 14:23:07 up 42 days,  3:17,  2 users,  load average: 0.52, 0.38, 0.31
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
admin    pts/0    192.168.1.100    12:30    0.00s  0.15s  0.01s w
deploy   pts/1    10.0.0.50        09:15    2:08m  0.32s  0.08s bash
```

| Column | Description |
|--------|-------------|
| `USER` | Username of logged-in user |
| `TTY` | Terminal (pts = remote, tty = local) |
| `FROM` | Source IP address or hostname |
| `LOGIN@` | When the user logged in |
| `IDLE` | How long since last activity |
| `JCPU` | Total CPU used by all processes on that tty |
| `PCPU` | CPU used by current process |
| `WHAT` | Currently running command |

```bash
# w options
$ w -h          # No header (skip uptime line)
$ w -s          # Short format (less detail)
$ w -i          # Show IP addresses instead of hostnames
$ w username    # Show only specific user
```

### Uptime in scripts — load monitoring
```bash
#!/bin/bash
# load_check.sh — Alert if load exceeds CPU count

CORES=$(nproc)
LOAD_1=$(awk '{print $1}' /proc/loadavg)

# Compare using bc (bash can't do float comparison)
OVERLOADED=$(echo "$LOAD_1 > $CORES" | bc -l)

if [ "$OVERLOADED" -eq 1 ]; then
    echo "⚠️ WARNING: Load ($LOAD_1) exceeds CPU cores ($CORES)!"
    echo "Top processes:"
    ps aux --sort=-%cpu | head -6
else
    echo "✅ System healthy: Load $LOAD_1 / $CORES cores"
fi
```

### Calculate uptime percentage
```bash
#!/bin/bash
# Calculate system idle percentage from /proc/uptime

read UPTIME IDLE < /proc/uptime
CORES=$(nproc)

# Idle time is cumulative across all cores
IDLE_PCT=$(echo "scale=2; ($IDLE / ($UPTIME * $CORES)) * 100" | bc)
BUSY_PCT=$(echo "scale=2; 100 - $IDLE_PCT" | bc)

echo "System Uptime:   $(uptime -p)"
echo "Total Uptime:    ${UPTIME}s"
echo "CPU Idle:        ${IDLE_PCT}%"
echo "CPU Busy (avg):  ${BUSY_PCT}%"
```

### Tracking reboot history
```bash
# Last reboot times (from wtmp)
$ last reboot | head -10
reboot   system boot  5.15.0-91-generic Sun Apr 13 11:06   still running
reboot   system boot  5.15.0-88-generic Fri Mar 21 09:42 - 11:05 (23+01:23)
reboot   system boot  5.15.0-88-generic Wed Feb 12 15:30 - 09:41 (36+18:11)

# Last shutdown times
$ last -x shutdown | head -5

# How many reboots in the last 30 days?
$ last reboot | grep -c "reboot"
```

---

## 🔴 Advanced Usage

### Server health monitoring script
```bash
#!/bin/bash
# server_health.sh — Comprehensive health check with uptime focus
# Usage: ./server_health.sh [--alert-load THRESHOLD]

THRESHOLD=${2:-$(nproc)}
CORES=$(nproc)

echo "╔══════════════════════════════════════════════╗"
echo "║         SERVER HEALTH REPORT                 ║"
echo "╠══════════════════════════════════════════════╣"
echo "║ Hostname:  $(hostname)"
echo "║ Date:      $(date '+%Y-%m-%d %H:%M:%S %Z')"
echo "║ Uptime:    $(uptime -p)"
echo "║ Boot Time: $(uptime -s)"
echo "║ CPU Cores: $CORES"
echo "╠══════════════════════════════════════════════╣"

# Load averages with status indicators
read L1 L5 L15 <<< $(awk '{print $1, $2, $3}' /proc/loadavg)

status_icon() {
    local load=$1
    local result=$(echo "$load > $CORES" | bc -l)
    if [ "$result" -eq 1 ]; then echo "🔴"; else
        result=$(echo "$load > ($CORES * 0.7)" | bc -l)
        if [ "$result" -eq 1 ]; then echo "🟡"; else echo "🟢"; fi
    fi
}

echo "║ Load Averages:"
echo "║   1 min:  $L1 $(status_icon $L1)"
echo "║   5 min:  $L5 $(status_icon $L5)"
echo "║  15 min:  $L15 $(status_icon $L15)"
echo "╠══════════════════════════════════════════════╣"

# Load per core ratio
RATIO=$(echo "scale=2; $L1 / $CORES" | bc)
echo "║ Load/Core Ratio: $RATIO"
if [ "$(echo "$RATIO > 1" | bc -l)" -eq 1 ]; then
    echo "║ ⚠️  SYSTEM OVERLOADED — processes queuing!"
elif [ "$(echo "$RATIO > 0.7" | bc -l)" -eq 1 ]; then
    echo "║ ⚡ High utilization — monitor closely"
else
    echo "║ ✅ System within normal parameters"
fi

echo "╠══════════════════════════════════════════════╣"

# Uptime security assessment
UPTIME_DAYS=$(awk '{printf "%d", $1/86400}' /proc/uptime)
echo "║ Uptime: $UPTIME_DAYS days"
if [ "$UPTIME_DAYS" -gt 90 ]; then
    echo "║ 🔴 CRITICAL: $UPTIME_DAYS days without reboot!"
    echo "║    Likely missing kernel security patches."
elif [ "$UPTIME_DAYS" -gt 30 ]; then
    echo "║ 🟡 WARNING: Consider patching & rebooting."
else
    echo "║ 🟢 Recently rebooted. Patch status likely current."
fi

echo "╠══════════════════════════════════════════════╣"
echo "║ Active Users: $(who | wc -l)"
who | while read line; do echo "║   $line"; done
echo "╚══════════════════════════════════════════════╝"
```

### Load average alerting with email notifications
```bash
#!/bin/bash
# load_alert.sh — Cron-based load monitoring with email alerts
# Add to crontab: */5 * * * * /opt/scripts/load_alert.sh

ADMIN_EMAIL="admin@example.com"
CORES=$(nproc)
ALERT_FILE="/tmp/load_alert_sent"

read L1 L5 L15 <<< $(awk '{print $1, $2, $3}' /proc/loadavg)

# Sustained high load: both 1m and 5m above threshold
HIGH_1=$(echo "$L1 > ($CORES * 2)" | bc -l)
HIGH_5=$(echo "$L5 > ($CORES * 1.5)" | bc -l)

if [ "$HIGH_1" -eq 1 ] && [ "$HIGH_5" -eq 1 ]; then
    if [ ! -f "$ALERT_FILE" ]; then
        BODY="⚠️ HIGH LOAD DETECTED on $(hostname)

Timestamp: $(date)
Uptime:    $(uptime)
Cores:     $CORES
Load:      $L1 / $L5 / $L15

Top CPU consumers:
$(ps aux --sort=-%cpu | head -11)

Memory:
$(free -h)

Disk:
$(df -h / | tail -1)"

        echo "$BODY" | mail -s "🔴 HIGH LOAD: $(hostname) - Load $L1" "$ADMIN_EMAIL"
        touch "$ALERT_FILE"
        logger -t load_alert "High load alert sent: $L1 $L5 $L15"
    fi
elif [ -f "$ALERT_FILE" ]; then
    echo "Load normalized on $(hostname): $L1 $L5 $L15" | \
        mail -s "🟢 RESOLVED: $(hostname) load normal" "$ADMIN_EMAIL"
    rm -f "$ALERT_FILE"
    logger -t load_alert "Load normalized: $L1 $L5 $L15"
fi
```

### Uptime tracking over time (CSV logging)
```bash
#!/bin/bash
# uptime_logger.sh — Log uptime & load to CSV for trend analysis
# Crontab: * * * * * /opt/scripts/uptime_logger.sh

LOGFILE="/var/log/uptime_history.csv"

# Create header if file doesn't exist
[ ! -f "$LOGFILE" ] && echo "timestamp,uptime_seconds,idle_seconds,load_1m,load_5m,load_15m,users,running_procs,total_procs" > "$LOGFILE"

TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
read UPTIME IDLE < /proc/uptime
read L1 L5 L15 PROCS LASTPID < /proc/loadavg
USERS=$(who | wc -l)
RUNNING=$(echo "$PROCS" | cut -d/ -f1)
TOTAL=$(echo "$PROCS" | cut -d/ -f2)

echo "$TIMESTAMP,$UPTIME,$IDLE,$L1,$L5,$L15,$USERS,$RUNNING,$TOTAL" >> "$LOGFILE"
```

### Programmatic uptime conversion
```bash
#!/bin/bash
# Convert /proc/uptime to every possible format

read TOTAL_SECONDS _ < /proc/uptime

DAYS=$((${TOTAL_SECONDS%.*} / 86400))
HOURS=$(( (${TOTAL_SECONDS%.*} % 86400) / 3600 ))
MINUTES=$(( (${TOTAL_SECONDS%.*} % 3600) / 60 ))
SECONDS=$((${TOTAL_SECONDS%.*} % 60))

echo "Raw seconds:  $TOTAL_SECONDS"
echo "Days:         $DAYS"
echo "Hours:        $((${TOTAL_SECONDS%.*} / 3600)) total hours"
echo "Pretty:       ${DAYS}d ${HOURS}h ${MINUTES}m ${SECONDS}s"
echo "Boot time:    $(uptime -s)"
echo "Epoch boot:   $(date -d "$(uptime -s)" +%s)"
```

---

## 🔒 Security & Pentesting

### Long Uptime = Missing Kernel Patches 🔴

```
High uptime is a sysadmin's pride — but a security engineer's nightmare.

Kernel updates REQUIRE a reboot. If uptime > 30 days, the system
is likely running an unpatched kernel with known vulnerabilities.
```

| Uptime Duration | Security Risk Level | Action Required |
|----------------|-------------------|-----------------|
| 0–7 days | 🟢 Low | Recently patched/rebooted |
| 7–30 days | 🟡 Moderate | Check for pending updates |
| 30–90 days | 🟠 High | Likely missing patches |
| 90+ days | 🔴 Critical | Almost certainly vulnerable |
| 365+ days | 💀 Extreme | Multiple unpatched CVEs guaranteed |

### CTF & Pentesting Reconnaissance 🎯
```bash
# On a compromised target, uptime gives valuable intel:
$ uptime
 14:23:07 up 387 days, 12:41, 1 user, load average: 0.05, 0.03, 0.01

# What this tells an attacker:
# 1. 387 days → Kernel NOT patched (search for kernel exploits!)
# 2. 1 user   → Low activity, might be unmonitored
# 3. Load 0.05 → Very idle system (good for running exploits)
# 4. Single user → Fewer eyes watching

# Recon workflow:
$ uname -r                    # Get exact kernel version
5.4.0-42-generic
$ uptime -s                   # Boot date
2025-05-05 01:42:15
# → Search: "Linux 5.4.0-42 exploit privilege escalation"
# → Check: DirtyPipe, DirtyCow, Sequoia, etc.
```

### Reboot analysis — incident response
```bash
# Was there an unexpected reboot?
$ last reboot | head -5
reboot   system boot  5.15.0-91   Sun Apr 13 11:06   still running
reboot   system boot  5.15.0-88   Fri Mar 21 09:42 - 11:05 (23+01:23)

# Check if kernel changed after reboot (indicates update)
# Old: 5.15.0-88 → New: 5.15.0-91 → Planned security update ✅
# Same version after reboot → Crash or manual restart ⚠️

# Cross-reference with system logs
$ journalctl --list-boots
$ journalctl -b -1 -p err      # Errors from previous boot
$ dmesg | grep -i "panic\|error\|crash"
```

### Detect if system uses live patching (no reboot needed)
```bash
# Check for kpatch / livepatch (Ubuntu/RHEL)
$ canonical-livepatch status 2>/dev/null || echo "No livepatch"
$ kpatch list 2>/dev/null || echo "No kpatch"

# If livepatch is active, high uptime may still be secure
# If not, high uptime = high risk
```

### Audit script for security teams
```bash
#!/bin/bash
# uptime_audit.sh — Security audit: uptime + patch status

echo "=== UPTIME SECURITY AUDIT ==="
echo "Hostname:     $(hostname)"
echo "Kernel:       $(uname -r)"
echo "Boot time:    $(uptime -s)"
echo "Uptime:       $(uptime -p)"

DAYS=$(awk '{printf "%d", $1/86400}' /proc/uptime)
echo "Days running: $DAYS"

echo ""
echo "--- Patch Status ---"
if command -v apt &>/dev/null; then
    SECURITY_UPDATES=$(apt list --upgradable 2>/dev/null | grep -c security)
    echo "Pending security updates: $SECURITY_UPDATES"
elif command -v yum &>/dev/null; then
    SECURITY_UPDATES=$(yum check-update --security 2>/dev/null | grep -c "\.x86_64\|\.noarch")
    echo "Pending security updates: $SECURITY_UPDATES"
fi

echo ""
echo "--- Reboot History (last 5) ---"
last reboot | head -5

echo ""
echo "--- Risk Assessment ---"
if [ "$DAYS" -gt 90 ]; then
    echo "🔴 CRITICAL: System has not been rebooted in $DAYS days!"
    echo "   Recommendation: Schedule immediate maintenance window."
elif [ "$DAYS" -gt 30 ]; then
    echo "🟡 WARNING: $DAYS days since last reboot."
    echo "   Recommendation: Review and apply pending patches."
else
    echo "🟢 OK: System rebooted $DAYS days ago."
fi
```

---

## 💡 Real World Pro Tips

### Tip 1: Load vs CPU Cores — The Only Ratio That Matters
```bash
# ALWAYS check load relative to CPU count
$ echo "Cores: $(nproc) | Load: $(awk '{print $1}' /proc/loadavg)"
Cores: 4 | Load: 3.82

# Quick health ratio
$ echo "scale=2; $(awk '{print $1}' /proc/loadavg) / $(nproc)" | bc
0.95
# → 0.95 means 95% utilized (per-core average)
# Below 1.0 = OK | Above 1.0 = Overloaded
```

### Tip 2: `uptime -s` + date math = exact uptime in any unit
```bash
# Exact uptime in hours
$ echo "scale=1; $(awk '{print $1}' /proc/uptime) / 3600" | bc
1009.0

# Days since boot (for scripting thresholds)
$ echo "$(( $(date +%s) - $(date -d "$(uptime -s)" +%s) ))" | awk '{print int($1/86400), "days"}'
42 days
```

### Tip 3: Use `w` instead of `uptime` + `who` separately
```bash
# w gives you everything in one command:
# - System uptime (header)
# - Who's logged in
# - What they're running
# - Where they're connecting from
$ w -i    # Include full IP addresses
```

### Tip 4: One-liner for Slack/webhook alerting
```bash
# Send uptime alert to Slack
LOAD=$(awk '{print $1}' /proc/loadavg)
CORES=$(nproc)
if (( $(echo "$LOAD > $CORES" | bc -l) )); then
    curl -s -X POST -H 'Content-type: application/json' \
        -d "{\"text\":\"🔴 HIGH LOAD on $(hostname): $LOAD (cores: $CORES)\"}" \
        "$SLACK_WEBHOOK_URL"
fi
```

### Tip 5: `/proc/uptime` is more accurate than `uptime -p`
```bash
# uptime -p rounds to minutes
$ uptime -p
up 42 days, 3 hours, 17 minutes

# /proc/uptime gives sub-second precision
$ cat /proc/uptime
3632257.42 14012853.68
# → 42 days, 3 hours, 17 minutes, 37.42 seconds
```

### Tip 6: Uptime in CTF recon reveals the time zone
```bash
# uptime shows the LOCAL time on the target system
$ uptime
 14:23:07 up 42 days, 3:17, ...
# → The target system's local time is 14:23:07
# → Compare with UTC to determine the timezone
# → Helps identify geographical location
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Instant — zero overhead | Load avg includes I/O wait (not just CPU) |
| Always available (no install needed) | Doesn't show per-core breakdown |
| Great scripting source (`/proc/uptime`) | Load spikes can be misleading without context |
| Combined info: time + uptime + load | No historical data (single snapshot) |
| `w` extends it with user details | Doesn't distinguish CPU vs I/O load |
| Works on every Linux/Unix system | Uptime resets if system crashes/reboots |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Quick server pulse check | `uptime` | One-line health overview |
| Human-readable uptime | `uptime -p` | Clean format for reports |
| When was last reboot? | `uptime -s` | Incident response / auditing |
| Scripted load monitoring | `cat /proc/loadavg` | Parseable, precise data |
| Who's logged in + uptime | `w` | Combined view |
| Exact seconds of uptime | `cat /proc/uptime` | Calculations, scripts |
| Reboot history | `last reboot` | Security audit trail |
| Load trend logging | Cron + `/proc/loadavg` | Capacity planning |
| CTF target recon | `uptime` + `uname -r` | Kernel exploit potential |
| Security patching audit | `uptime -s` + `apt` | Compliance reporting |

---

## ⚠️ Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| "Load 4.0 = bad!" | Only bad if cores < 4 | Always compare to `nproc` |
| Ignoring I/O in load avg | Disk-heavy workloads inflate load | Use `iostat` / `vmstat` to separate |
| Treating high uptime as good | Missing kernel patches = vulnerable | Reboot after security updates |
| Using `uptime` for CPU % | Load avg ≠ CPU percentage | Use `top`, `mpstat`, or `sar` |
| Parsing `uptime` output with regex | Format varies by uptime duration | Use `/proc/uptime` for scripts |
| Not checking `last reboot` | Uptime only shows current boot | `last reboot` shows full history |
| Confusing users count | Counts sessions, not unique users | One user with 3 terminals = 3 users |

### Format variation trap:
```bash
# uptime output format CHANGES based on duration:
 10:30:01 up  3:42,   1 user,  ...    # Hours only
 10:30:01 up  2 days, 3:42,  1 user,  ...    # Days + hours
 10:30:01 up 42 days, 3:42,  2 users, ...    # Longer duration

# This makes regex parsing fragile!
# ✅ Always use /proc/uptime or /proc/loadavg in scripts
```

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Run `uptime` and identify each component of the output
2. Use `uptime -p` to see your uptime in human-readable format
3. Use `uptime -s` to find when your system was last booted
4. Read `/proc/uptime` and convert the first value to days/hours/minutes manually
5. Check how many CPU cores you have with `nproc` and relate that to your load average

### 🟡 Intermediate
6. Write a script that prints "OVERLOADED" if load average (1 min) exceeds the number of CPU cores
7. Use the `w` command to see who's logged in along with uptime — identify each column
8. Read `/proc/loadavg` and explain what each field means (including the process count)
9. Use `last reboot` to check your system's reboot history and correlate with `uptime -s`
10. Write a one-liner that calculates your system's idle percentage from `/proc/uptime`

### 🔴 Advanced
11. Create a cron job that logs uptime and load averages to a CSV file every minute
12. Write a server health script that:
    - Shows uptime, load, users
    - Alerts if load/core ratio > 1.0
    - Warns if uptime > 30 days (patching risk)
    - Lists top 5 CPU-consuming processes
13. On a CTF box: use `uptime`, `uname -r`, and `last reboot` to assess how likely the kernel is exploitable
14. Build a monitoring dashboard one-liner that outputs JSON with uptime, load, and user data for API consumption:
    ```bash
    echo "{\"host\":\"$(hostname)\",\"uptime_sec\":$(awk '{print $1}' /proc/uptime),\"load\":[$(awk '{printf "%s,%s,%s",$1,$2,$3}' /proc/loadavg)],\"users\":$(who|wc -l)}"
    ```

---

## 🧠 Cheat Sheet

```
BASIC:
  uptime              → Full output: time, uptime, users, load
  uptime -p           → Pretty: "up 42 days, 3 hours, 17 minutes"
  uptime -s           → Since: "2026-04-13 11:06:22"

RAW DATA:
  cat /proc/uptime    → "3632257.42 14012853.68" (uptime + idle secs)
  cat /proc/loadavg   → "0.52 0.38 0.31 2/487 19832" (load + procs)

RELATED COMMANDS:
  w                   → uptime + who's logged in + what they're doing
  w -i                → w with full IP addresses
  last reboot         → Reboot history from wtmp
  nproc               → Number of CPU cores

LOAD AVERAGE INTERPRETATION:
  Load ≤ Cores        → ✅ Healthy
  Load > Cores        → ⚠️ Overloaded (processes queuing)
  Load / Cores < 0.7  → 🟢 Comfortable
  Load / Cores > 1.0  → 🔴 Saturated

QUICK SCRIPTS:
  # Load per core ratio
  echo "scale=2; $(awk '{print $1}' /proc/loadavg)/$(nproc)" | bc

  # Days since boot
  awk '{printf "%d days\n", $1/86400}' /proc/uptime

  # Overload check
  [ $(echo "$(awk '{print $1}' /proc/loadavg) > $(nproc)" | bc) -eq 1 ] && echo "OVERLOADED!"

SECURITY:
  uptime > 30 days    → Check for kernel updates!
  uptime > 90 days    → Almost certainly unpatched
  last reboot         → Verify reboots were planned
  canonical-livepatch status  → Check for live kernel patches
```

---

> **Previous**: [`dmesg` ←](./52_dmesg.md) | **Next**: [`free` →](./54_free.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
