# 🛠️ `kill` & `killall` — Terminate Processes | Linux Master Note

> **When a process misbehaves, you need to put it down. `kill` sends signals to processes — from gentle "please stop" to brutal "die immediately".**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--unix-signals)
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

### The kill family:
| Command | Description |
|---------|-------------|
| `kill` | Send signal to a process by **PID** |
| `killall` | Kill processes by **name** |
| `pkill` | Kill processes by **pattern** (regex) |
| `xkill` | Click on a window to kill it (GUI) |

### Key concept:
> `kill` doesn't just "kill" — it **sends signals**. Killing is just one type of signal. You can also pause, resume, reload, and more!

---

## 📖 Theory — Unix Signals

### What are signals?
Signals are **software interrupts** — messages sent to a process to tell it to do something. The kernel delivers the signal, and the process has a **signal handler** to respond.

### Essential Signals:

| # | Signal | Name | Default Action | Description |
|---|--------|------|---------------|-------------|
| `1` | `SIGHUP` | Hangup | Terminate | Terminal closed / reload config |
| `2` | `SIGINT` | Interrupt | Terminate | `Ctrl+C` from keyboard |
| `3` | `SIGQUIT` | Quit | Core dump | `Ctrl+\` from keyboard |
| `9` | `SIGKILL` | Kill | **Force kill** | ⚡ Cannot be caught or ignored! |
| `15` | `SIGTERM` | Terminate | Graceful exit | **Default signal** — process can clean up |
| `18` | `SIGCONT` | Continue | Resume | Resume a stopped process |
| `19` | `SIGSTOP` | Stop | **Pause** | Cannot be caught (like SIGKILL) |
| `20` | `SIGTSTP` | Terminal Stop | Pause | `Ctrl+Z` from keyboard |

### Signal handling rules:
- **SIGKILL (9)** and **SIGSTOP (19)** — **CANNOT** be caught, blocked, or ignored
- **SIGTERM (15)** — Process CAN catch it and perform cleanup (save files, close connections)
- **SIGHUP (1)** — Many daemons reload config instead of dying

### Kill order best practice:
```
Step 1: kill PID          → SIGTERM (15) — ask nicely
Step 2: wait 5 seconds
Step 3: kill -9 PID       → SIGKILL (9) — force kill (last resort!)
```

> ⚠️ SIGKILL (9) skips cleanup — data may be lost, temp files left behind, locks not released!

---

## 🧰 Syntax & Options

### `kill` syntax:
```bash
kill [OPTIONS] PID(s)...
```

| Flag | Description |
|------|-------------|
| `-SIGNAL` or `-s SIGNAL` | Specify signal (number or name) |
| `-l` | List all signal names |
| `-L` | List signals in table format |

### `killall` syntax:
```bash
killall [OPTIONS] NAME
```

| Flag | Description |
|------|-------------|
| `-SIGNAL` | Specify signal |
| `-i` | Interactive — ask before killing each |
| `-u USER` | Kill only processes owned by user |
| `-w` | Wait for processes to die |
| `-r` | Use regex for name matching |
| `-e` | Exact match (long names) |
| `-v` | Verbose |
| `-I` | Case-insensitive name match |

### `pkill` syntax:
```bash
pkill [OPTIONS] PATTERN
```

| Flag | Description |
|------|-------------|
| `-SIGNAL` | Specify signal |
| `-u USER` | Match by user |
| `-f` | Match against full command line |
| `-x` | Exact name match |
| `-t TTY` | Match by terminal |
| `-P PPID` | Match by parent PID |
| `-n` | Newest matching process only |
| `-o` | Oldest matching process only |

---

## 🟢 Basic Usage

```bash
# List all signals
$ kill -l
 1) SIGHUP    2) SIGINT    3) SIGQUIT   9) SIGKILL
15) SIGTERM  18) SIGCONT  19) SIGSTOP  20) SIGTSTP

# Kill by PID (graceful — SIGTERM)
$ kill 1234

# Kill by PID (force — SIGKILL)
$ kill -9 1234
$ kill -SIGKILL 1234
$ kill -KILL 1234         # All equivalent

# Kill by name
$ killall firefox
$ killall python3

# Kill by pattern
$ pkill -f "python3 app.py"

# Interrupt (same as Ctrl+C)
$ kill -2 1234
$ kill -SIGINT 1234
```

---

## 🟡 Intermediate Usage

### Find PID first, then kill
```bash
# Using pgrep
$ pgrep nginx
789 790 791
$ kill 789 790 791

# Using pidof
$ kill $(pidof nginx)

# Using ps + grep
$ ps aux | grep "[n]ginx" | awk '{print $2}' | xargs kill
```

### Reload daemon config (SIGHUP)
```bash
# Reload nginx config without stopping
$ sudo kill -HUP $(pgrep -o nginx)
# OR
$ sudo kill -1 $(cat /var/run/nginx.pid)
# OR
$ sudo systemctl reload nginx     # modern way
```

### Kill all processes by user
```bash
# Kill all processes owned by 'alice'
$ sudo pkill -u alice

# Kill all processes for user (forcefully)
$ sudo pkill -9 -u alice

# Using killall
$ sudo killall -u alice -9
```

### Interactive kill (ask before each)
```bash
$ killall -i python3
Kill python3(1234) ? (y/N) y
Kill python3(5678) ? (y/N) n
```

### Wait for process to die
```bash
$ killall -w nginx     # Waits until all nginx processes are gone
```

### Kill by terminal
```bash
# Kill all processes on pts/2 (someone's SSH session)
$ sudo pkill -t pts/2
```

---

## 🔴 Advanced Usage

### Graceful shutdown sequence
```bash
#!/bin/bash
# graceful_kill.sh — Proper kill order
PID=$1

echo "Sending SIGTERM to $PID..."
kill $PID 2>/dev/null

# Wait up to 10 seconds for graceful shutdown
for i in $(seq 1 10); do
    if ! kill -0 $PID 2>/dev/null; then
        echo "Process $PID stopped gracefully."
        exit 0
    fi
    sleep 1
done

echo "Process still running. Sending SIGKILL..."
kill -9 $PID 2>/dev/null
echo "Process $PID force killed."
```

### Kill process tree (parent + all children)
```bash
# Kill parent and all child processes
$ pkill -P 1234          # Kill children of PID 1234
$ kill 1234              # Then kill parent

# Or use process group kill
$ kill -- -$(ps -o pgid= -p 1234 | tr -d ' ')
# Negative PID = kill entire process group
```

### Security — Kill Suspicious Processes 🔒
```bash
# Kill all processes running from /tmp (potential malware)
$ sudo lsof +D /tmp/ | awk 'NR>1 {print $2}' | sort -u | xargs sudo kill -9

# Kill processes with deleted executables
$ sudo ls -la /proc/*/exe 2>/dev/null | grep deleted | awk -F/ '{print $3}' | xargs sudo kill -9

# Kill all reverse shells
$ sudo pkill -9 -f "bash -i"
$ sudo pkill -9 -f "nc.*-e"
$ sudo pkill -9 -f "ncat.*-e"

# Kill cryptominers
$ sudo pkill -9 -f "xmrig\|minerd\|cryptonight"
```

### Check if process exists (scripting)
```bash
# kill -0 sends NO signal — just checks if process exists
$ kill -0 1234 2>/dev/null && echo "Running" || echo "Not running"

# Use in scripts
if kill -0 "$PID" 2>/dev/null; then
    echo "Process $PID is still running"
fi
```

### Freeze/Resume processes
```bash
# Pause a process (SIGSTOP — cannot be caught!)
$ kill -STOP 1234
$ kill -19 1234

# Resume a paused process (SIGCONT)
$ kill -CONT 1234
$ kill -18 1234

# Use case: pause heavy process during peak hours
$ kill -STOP $(pgrep -f backup_script)
# ... later ...
$ kill -CONT $(pgrep -f backup_script)
```

---

## 🔗 Piping & Combining

```bash
# Kill all matching processes
$ pgrep -f "python3 app" | xargs kill

# Kill oldest process matching pattern
$ pkill -o -f "worker"

# Kill newest process matching pattern
$ pkill -n -f "worker"

# Find and kill with one command
$ ps aux | grep "[z]ombie" | awk '{print $2}' | xargs kill -9

# Kill all except specific PID
$ pgrep python3 | grep -v 1234 | xargs kill

# Kill processes using more than 50% CPU
$ ps -eo pid,%cpu --no-headers | awk '$2 > 50 {print $1}' | xargs kill
```

---

## 💡 Real World Pro Tips

### Tip 1: Always SIGTERM before SIGKILL
```bash
# ✅ Correct order
$ kill 1234         # SIGTERM first (cleanup chance)
$ sleep 5
$ kill -9 1234      # SIGKILL only if still running

# ❌ Bad practice
$ kill -9 1234      # Immediate — no cleanup!
```

### Tip 2: Use `pkill` instead of `ps | grep | kill`
```bash
# ❌ Long way
$ ps aux | grep nginx | grep -v grep | awk '{print $2}' | xargs kill

# ✅ Short way
$ pkill nginx
```

### Tip 3: Kill with `kill -0` check first
```bash
if kill -0 "$PID" 2>/dev/null; then
    kill "$PID"
    echo "Killed $PID"
else
    echo "Process $PID not found"
fi
```

### Tip 4: Use `timeout` to prevent hanging
```bash
# Run command with timeout — auto-kills if too slow
$ timeout 30 ./slow_script.sh
# Sends SIGTERM after 30 seconds

$ timeout -s KILL 30 ./very_slow_script.sh
# Sends SIGKILL after 30 seconds
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Multiple signal types | SIGKILL skips cleanup |
| kill/killall/pkill options | Can kill wrong process |
| Process group kill | No undo — process is gone |
| Check existence with kill -0 | Need PID or name |
| SIGHUP for config reload | Zombies can't be killed (kill parent!) |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Kill specific PID | `kill PID` | Precise |
| Kill by name | `killall name` | Convenient |
| Kill by pattern | `pkill -f "pattern"` | Flexible |
| Reload config | `kill -HUP PID` | No downtime |
| Force kill frozen | `kill -9 PID` | Last resort |
| Pause process | `kill -STOP PID` | Temporary freeze |
| Kill user's processes | `pkill -u user` | User management |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `kill -9` as first choice | Try `kill` (SIGTERM) first |
| Killing wrong PID | Double-check with `ps -p PID` |
| Can't kill zombie process | Kill the PARENT process instead |
| `killall` behaving differently | On Solaris it kills ALL processes! Use only on Linux |
| Not checking if process exists | Use `kill -0 PID` first |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Start `sleep 300 &` and kill it with `kill`
2. List all available signals with `kill -l`
3. Use `killall` to kill a process by name

### 🟡 Intermediate
4. Write a graceful kill script (SIGTERM → wait → SIGKILL)
5. Pause and resume a process with SIGSTOP/SIGCONT
6. Use `pkill -u` to kill all processes by a user

### 🔴 Advanced
7. Kill an entire process tree (parent + children)
8. Find and kill processes running from /tmp
9. Use `kill -0` to check process existence in a monitoring script

---

## 🧠 Cheat Sheet

```
SIGNALS:
  1=SIGHUP (reload)   2=SIGINT (Ctrl+C)   9=SIGKILL (force)
  15=SIGTERM (default) 18=SIGCONT (resume)  19=SIGSTOP (pause)

KILL BY PID:
  kill PID            → Graceful (SIGTERM)
  kill -9 PID         → Force (SIGKILL)
  kill -HUP PID       → Reload config
  kill -STOP PID      → Pause process
  kill -CONT PID      → Resume process
  kill -0 PID         → Check if alive

KILL BY NAME:
  killall name        → Kill all by name
  killall -i name     → Interactive
  killall -u user     → By user

KILL BY PATTERN:
  pkill pattern       → By name pattern
  pkill -f "full cmd" → By full command
  pkill -u user       → By user
  pkill -P ppid       → By parent PID

BEST PRACTICE:
  kill PID → sleep 5 → kill -9 PID (if still alive)
```

---

> **Previous**: [`top/htop` ←](./28_top_htop.md) | **Next**: [`bg/fg/jobs` →](./30_bg_fg_jobs.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
