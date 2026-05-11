# 🛠️ `tail` — View Last Lines of File | Linux Master Note

> **Monitor live logs, check file endings, and follow real-time output. `tail -f` is every sysadmin's best friend.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Syntax & Options](#-syntax--options)
3. [Basic Usage](#-basic-usage)
4. [Intermediate Usage](#-intermediate-usage)
5. [Advanced Usage](#-advanced-usage)
6. [Piping & Combining](#-piping--combining)
7. [Real World Pro Tips](#-real-world-pro-tips)
8. [Pros & Cons](#-pros--cons)
9. [Where & When to Use](#-where--when-to-use)
10. [Practice Exercises](#-practice-exercises)
11. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

`tail` prints the **last N lines** (default: 10) of a file. Its killer feature is **`-f` (follow)** — real-time log monitoring that continuously shows new lines as they're appended.

---

## 🧰 Syntax & Options

```bash
tail [OPTIONS] [FILE(s)...]
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-n NUM` | `--lines=NUM` | Print last NUM lines |
| `-n +NUM` | — | Print from line NUM onwards |
| `-c NUM` | `--bytes=NUM` | Print last NUM bytes |
| `-f` | `--follow` | Follow file — show new lines in real-time |
| `-F` | `--follow --retry` | Follow + retry if file is recreated (log rotation) |
| `--pid=PID` | — | Stop following when process PID dies |
| `-s NUM` | `--sleep-interval=NUM` | Check interval in seconds (default 1) |
| `-q` | `--quiet` | Don't print filename headers |
| `-v` | `--verbose` | Always print filename headers |

---

## 🟢 Basic Usage

```bash
# Last 10 lines (default)
$ tail /var/log/syslog

# Last 20 lines
$ tail -n 20 /var/log/syslog
$ tail -20 /var/log/syslog

# Last 100 bytes
$ tail -c 100 file.txt
```

---

## 🟡 Intermediate Usage

### Print from line N onwards
```bash
# Skip header, print from line 2 onwards
$ tail -n +2 data.csv    # Skip CSV header

# Print from line 100 to end
$ tail -n +100 largefile.txt
```

### Follow mode — live log monitoring 🔥
```bash
# Follow a log file (Ctrl+C to stop)
$ tail -f /var/log/syslog

# Follow with retry (handles log rotation)
$ tail -F /var/log/auth.log

# Follow multiple files
$ tail -f /var/log/syslog /var/log/auth.log
```

### Follow until process ends
```bash
# Follow log until process 1234 dies
$ tail -f --pid=1234 app.log
```

---

## 🔴 Advanced Usage

### Monitor specific patterns in real-time
```bash
# Follow + grep for errors only
$ tail -f /var/log/syslog | grep --line-buffered "ERROR"

# Follow + colorize matches
$ tail -f /var/log/auth.log | grep --color=always -i "fail"

# Follow + filter + timestamp
$ tail -f /var/log/syslog | grep --line-buffered "ssh" | awk '{print strftime("%H:%M:%S"), $0}'
```

### Extract specific line ranges
```bash
# Lines 50-60 (combine head + tail)
$ head -60 file.txt | tail -11
# OR
$ sed -n '50,60p' file.txt
```

### Intrusion detection — monitor auth logs
```bash
# Watch for brute force attacks in real-time
$ tail -f /var/log/auth.log | grep --line-buffered "Failed password"

# Monitor SSH connections
$ tail -f /var/log/auth.log | grep --line-buffered "sshd"

# Alert on sudo usage
$ tail -f /var/log/auth.log | grep --line-buffered "sudo"
```

### Multi-file monitoring with `multitail`
```bash
$ sudo apt install multitail
$ multitail /var/log/syslog /var/log/auth.log /var/log/nginx/access.log
# Split-screen real-time monitoring!
```

---

## 🔗 Piping & Combining

```bash
# Last 5 processes by CPU
$ ps aux --sort=-%cpu | tail -n +2 | head -5

# Last 10 modified files
$ ls -lt | tail -n +2 | head -10

# Recent history commands
$ history | tail -20

# Last error in log
$ grep "ERROR" /var/log/app.log | tail -1
```

---

## 💡 Real World Pro Tips

### Tip 1: `-F` vs `-f`
```bash
$ tail -f file.log     # Stops if file is deleted/rotated
$ tail -F file.log     # Retries — handles logrotate properly!
# ALWAYS use -F for production log monitoring
```

### Tip 2: Line-buffered grep for real-time filtering
```bash
# Without --line-buffered, grep buffers output (delayed!)
$ tail -f log | grep --line-buffered "pattern"
```

### Tip 3: Quick server debug workflow
```bash
# Terminal 1: Follow application log
$ tail -F /var/log/app/error.log

# Terminal 2: Follow access log
$ tail -F /var/log/nginx/access.log

# Terminal 3: Follow system log
$ tail -F /var/log/syslog | grep --line-buffered "$(hostname)"
```

### Tip 4: Log analysis one-liners
```bash
# Count errors in last 100 lines
$ tail -100 /var/log/app.log | grep -c "ERROR"

# Last failed login attempt
$ tail -100 /var/log/auth.log | grep "Failed" | tail -1
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| `-f` follow is incredibly useful | Can't scroll backward |
| `-F` handles log rotation | No search/navigation |
| Fast — reads from end of file | No interactive mode |
| Essential for debugging | Buffer issues with pipes |

---

## 📍 Where & When to Use

| Scenario | Use `tail`? | Alternative |
|----------|-----------|-------------|
| Check recent log entries | ✅ Yes | — |
| Live log monitoring | ✅ `tail -F` | `less +F`, `multitail` |
| Skip file header | ✅ `tail -n +2` | `sed 1d` |
| File beginning | ❌ No | `head` |
| Full file browse | ❌ No | `less` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. View last 10 lines of `/var/log/syslog`
2. View last 20 lines of a file
3. Follow a log file in real-time

### 🟡 Intermediate
4. Skip CSV header using `tail -n +2`
5. Follow a log and filter for "error"
6. Monitor multiple log files simultaneously

### 🔴 Advanced
7. Set up a real-time SSH attack monitor with `tail -F | grep`
8. Extract lines 50-100 from a file
9. Use `--pid` to follow until a process ends
10. Install and use `multitail` for split-screen monitoring

---

## 🧠 Cheat Sheet

```
tail file           → Last 10 lines       tail -f file    → Follow (live)
tail -n 20 file     → Last 20 lines       tail -F file    → Follow + retry
tail -n +2 file     → From line 2 on      tail -c 100     → Last 100 bytes
tail -f log | grep --line-buffered "ERROR"  → Live error filter

⭐ ALWAYS use -F (not -f) for production log monitoring!
```

---

> **Previous**: [`head` ←](./13_head.md) | **Next**: [`grep` →](./15_grep.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
