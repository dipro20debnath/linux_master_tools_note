# 🛠️ `nohup` — Run Command Immune to Hangups | Linux Master Note

> **Keep your processes alive even after you disconnect. `nohup` ensures your long-running tasks survive terminal closure and SSH disconnections.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--sighup--terminal-lifecycle)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [Alternatives](#-alternatives--screen-tmux-systemd)
8. [Real World Pro Tips](#-real-world-pro-tips)
9. [Pros & Cons](#-pros--cons)
10. [Where & When to Use](#-where--when-to-use)
11. [Common Mistakes](#-common-mistakes)
12. [Practice Exercises](#-practice-exercises)
13. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### What is `nohup`?
`nohup` (**No H**ang**Up**) runs a command that is **immune to the SIGHUP signal**. When you close your terminal or disconnect from SSH, all child processes receive SIGHUP and die — `nohup` prevents this.

### The problem it solves:
```
You: SSH into server → Start long task → Close laptop → Task DIES! 😱

With nohup:
You: SSH into server → nohup task & → Close laptop → Task CONTINUES! ✅
```

### When you need it:
- Long-running downloads, uploads
- Database migrations
- Machine learning training
- Server maintenance scripts
- Any task that takes longer than your SSH session

---

## 📖 Theory — SIGHUP & Terminal Lifecycle

### What happens when terminal closes:
```
1. You close terminal / SSH disconnects
2. Kernel sends SIGHUP (signal 1) to the shell
3. Shell sends SIGHUP to ALL child processes
4. Child processes terminate (default SIGHUP handler)
5. Your long-running task is DEAD
```

### How `nohup` prevents this:
```
1. nohup sets the SIGHUP handler to SIG_IGN (ignore)
2. When terminal closes → SIGHUP sent → IGNORED
3. Process continues running
4. Output redirected to nohup.out (since terminal is gone)
```

### `nohup` vs `disown` vs `setsid`:
| Tool | Mechanism | Use Case |
|------|-----------|----------|
| `nohup` | Ignores SIGHUP before start | Plan ahead — start immune |
| `disown` | Removes job from shell's table | Already running — detach now |
| `setsid` | Creates new session (new SID) | Complete session detachment |
| `screen/tmux` | Virtual terminal | Full terminal persistence |

---

## 🧰 Syntax & Options

```bash
nohup COMMAND [ARGS...] &
```

- `nohup` has **no flags** — it's ultra-simple
- Always use `&` at the end to run in background
- Output goes to `nohup.out` in current directory (or `$HOME/nohup.out`)

### Default behavior:
| Aspect | Behavior |
|--------|----------|
| SIGHUP | Ignored |
| stdout | Redirected to `nohup.out` |
| stderr | Redirected to `nohup.out` |
| stdin | Redirected from `/dev/null` |
| Working directory | Stays the same |
| Exit code | Same as the command's |

---

## 🟢 Basic Usage

```bash
# Basic — run immune to hangups
$ nohup python3 train_model.py &
[1] 12345
nohup: ignoring input and appending output to 'nohup.out'

# Check output
$ tail -f nohup.out

# Check if still running
$ jobs -l
$ ps aux | grep train_model

# Custom output file
$ nohup python3 train_model.py > training.log 2>&1 &

# Discard all output
$ nohup python3 task.py > /dev/null 2>&1 &
```

---

## 🟡 Intermediate Usage

### Redirect output properly
```bash
# Separate stdout and stderr
$ nohup command > output.log 2> error.log &

# Append to existing log
$ nohup command >> output.log 2>&1 &

# With timestamp in filename
$ nohup ./backup.sh > "backup_$(date +%Y%m%d_%H%M%S).log" 2>&1 &
```

### Combine with `disown`
```bash
# Start, background, and fully detach
$ nohup long_task > task.log 2>&1 &
$ disown
# Now completely detached — safe to close terminal

# Already running a foreground task? (forgot nohup!)
$ python3 big_task.py        # Already started!
# Press Ctrl+Z
[1]+  Stopped    python3 big_task.py
$ bg %1
$ disown %1                  # Detach (but no SIGHUP protection)
```

### Run multiple nohup commands
```bash
# Multiple tasks
$ nohup ./task1.sh > task1.log 2>&1 &
$ nohup ./task2.sh > task2.log 2>&1 &
$ nohup ./task3.sh > task3.log 2>&1 &

# Check all
$ jobs -l
[1]  12345 Running    nohup ./task1.sh ...
[2]  12346 Running    nohup ./task2.sh ...
[3]  12347 Running    nohup ./task3.sh ...
```

### Monitor nohup process
```bash
# Watch log in real-time
$ tail -f nohup.out

# Check process status
$ ps -p 12345 -o pid,%cpu,%mem,etime,cmd

# Watch with refresh
$ watch -n 5 'ps -p 12345 -o pid,%cpu,%mem,etime,cmd'
```

---

## 🔴 Advanced Usage

### Server Deployment Script
```bash
#!/bin/bash
# deploy_and_run.sh
echo "Starting deployment..."
cd /opt/myapp

# Stop old instance
pkill -f "python3 app.py" 2>/dev/null
sleep 2

# Pull latest code
git pull origin main

# Start new instance
nohup python3 app.py > /var/log/myapp/app.log 2>&1 &
echo $! > /var/run/myapp.pid
echo "App started with PID $(cat /var/run/myapp.pid)"
```

### Long-Running Security Scan
```bash
# Start nmap scan that survives SSH disconnect
$ nohup nmap -sV -sC -p- -oA full_scan 192.168.1.0/24 > scan_progress.log 2>&1 &
$ echo $! > scan.pid
$ echo "Scan PID: $(cat scan.pid)"

# Reconnect later and check
$ tail -f scan_progress.log
$ kill -0 $(cat scan.pid) && echo "Still running" || echo "Done"
```

### Wrapper Script with Notifications
```bash
#!/bin/bash
# nohup_notify.sh — Run task and notify when done
TASK="$@"
LOG="task_$(date +%s).log"

echo "Starting: $TASK"
echo "Log: $LOG"
echo "PID: $$"

# Run the task
eval "$TASK" > "$LOG" 2>&1
EXIT_CODE=$?

# Notify (email, webhook, etc.)
if [ $EXIT_CODE -eq 0 ]; then
    echo "SUCCESS: $TASK completed" | mail -s "Task Done" admin@example.com
else
    echo "FAILED (exit $EXIT_CODE): $TASK" | mail -s "Task Failed!" admin@example.com
fi

# Usage:
# nohup ./nohup_notify.sh python3 train_model.py &
```

### Process persistence check
```bash
#!/bin/bash
# ensure_running.sh — Restart if process dies
COMMAND="python3 /opt/myapp/app.py"
PIDFILE="/var/run/myapp.pid"
LOGFILE="/var/log/myapp/app.log"

while true; do
    if [ -f "$PIDFILE" ] && kill -0 $(cat "$PIDFILE") 2>/dev/null; then
        sleep 30    # Check every 30 seconds
    else
        echo "$(date): Restarting $COMMAND" >> "$LOGFILE"
        nohup $COMMAND >> "$LOGFILE" 2>&1 &
        echo $! > "$PIDFILE"
    fi
done

# Run this watchdog itself with nohup:
# nohup ./ensure_running.sh > /var/log/watchdog.log 2>&1 &
```

---

## 🔄 Alternatives — screen, tmux, systemd

### `screen` — Terminal multiplexer
```bash
$ screen -S mysession          # Create named session
$ python3 long_task.py         # Run your task
# Press Ctrl+A, D to detach

$ screen -r mysession          # Reattach later
$ screen -ls                   # List sessions
```

### `tmux` — Modern terminal multiplexer (RECOMMENDED)
```bash
$ tmux new -s work             # Create session
$ python3 long_task.py         # Run task
# Press Ctrl+B, D to detach

$ tmux attach -t work          # Reattach
$ tmux ls                      # List sessions
```

### `systemd` — For permanent services
```bash
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=dipro
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

$ sudo systemctl enable myapp
$ sudo systemctl start myapp
```

### When to use what:
| Tool | Best For |
|------|----------|
| `nohup` | Quick one-off tasks |
| `disown` | Already started, forgot nohup |
| `screen` | Need to reattach to output |
| `tmux` | Multiple windows, modern features |
| `systemd` | Production services, auto-restart |

---

## 💡 Real World Pro Tips

### Tip 1: Always redirect output!
```bash
# ❌ Output goes to nohup.out (might fill disk!)
$ nohup heavy_task &

# ✅ Redirect to specific log
$ nohup heavy_task > /var/log/task.log 2>&1 &
```

### Tip 2: Save the PID!
```bash
$ nohup task > log.txt 2>&1 &
$ echo $! > task.pid
# Later:
$ kill $(cat task.pid)
```

### Tip 3: The full detachment combo
```bash
# The bulletproof way to start a persistent background task:
$ nohup command > output.log 2>&1 &
$ disown
# This is immune to SIGHUP + removed from shell's job table
```

### Tip 4: Check nohup.out size
```bash
# nohup.out can grow huge! Monitor it:
$ ls -lh nohup.out
$ du -h nohup.out

# Use logrotate for long-running processes
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Ultra-simple syntax | Can't reattach to see output live |
| No installation needed | nohup.out can grow huge |
| Works with any command | No session management |
| Survives terminal close | Can't interact after detaching |
| Light on resources | Not suitable for production services |

---

## 📍 Where & When to Use

| Scenario | Tool | Why |
|----------|------|-----|
| Quick background task | `nohup cmd &` | Simple, fast |
| Need to reattach later | `tmux` or `screen` | Session persistence |
| Already running task | `Ctrl+Z` + `bg` + `disown` | Rescue |
| Production service | `systemd` | Auto-restart, logging |
| ML training over SSH | `nohup` or `tmux` | Long-running |
| Security scan overnight | `nohup nmap ... &` | Survives disconnect |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `&` at the end | `nohup cmd &` — always add `&`! |
| Not redirecting output | `nohup cmd > log 2>&1 &` |
| nohup.out fills up disk | Redirect or `/dev/null` |
| Losing track of PID | `echo $! > pidfile` |
| Using for production services | Use `systemd` instead |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Run `nohup sleep 120 &` and verify with `ps`
2. Check the contents of `nohup.out`
3. Kill the nohup process using saved PID

### 🟡 Intermediate
4. Run a task with custom output redirect
5. Use `nohup` + `disown` for full detachment
6. Monitor nohup process with `tail -f` and `watch`

### 🔴 Advanced
7. Write a deployment script using nohup
8. Create a watchdog script that restarts crashed processes
9. Compare nohup vs tmux for a long-running scan

---

## 🧠 Cheat Sheet

```
BASIC:
  nohup command &                      → Run immune to hangup
  nohup command > log.txt 2>&1 &       → Custom output
  nohup command > /dev/null 2>&1 &     → Discard output

TRACK:
  echo $!                              → Last background PID
  echo $! > task.pid                   → Save PID
  kill $(cat task.pid)                 → Kill later

FULL DETACH:
  nohup cmd > log 2>&1 & disown       → Bulletproof combo

MONITOR:
  tail -f log.txt                      → Watch output
  ps -p PID -o pid,%cpu,%mem,etime     → Process status
  kill -0 PID && echo "alive"          → Check if running

ALTERNATIVES:
  tmux new -s name → Ctrl+B,D → tmux attach -t name
  screen -S name → Ctrl+A,D → screen -r name
  systemd → for production services
```

---

> **Previous**: [`bg/fg/jobs` ←](./30_bg_fg_jobs.md) | **Next**: [`cron/crontab` →](./32_cron_crontab.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
