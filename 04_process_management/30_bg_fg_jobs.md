# 🛠️ `bg`, `fg` & `jobs` — Job Control | Linux Master Note

> **Run multiple tasks in one terminal. Background, foreground, pause, resume — master job control and become a multitasking wizard.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--job-control-model)
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

### What is Job Control?
Job control lets you **manage multiple processes** within a single terminal session — run tasks in the background, bring them to the foreground, pause and resume them.

### The Three Commands:
| Command | Purpose |
|---------|---------|
| `jobs` | List all background/stopped jobs |
| `bg` | Resume a stopped job in the **background** |
| `fg` | Bring a background/stopped job to the **foreground** |

### Key Keyboard Shortcuts:
| Shortcut | Signal | Action |
|----------|--------|--------|
| `Ctrl+C` | SIGINT (2) | **Kill** foreground process |
| `Ctrl+Z` | SIGTSTP (20) | **Pause/Stop** foreground process |
| `Ctrl+\` | SIGQUIT (3) | Kill + core dump |

---

## 📖 Theory — Job Control Model

### Foreground vs Background:
```
Terminal
├── Foreground job (blocks terminal — you wait)
│   └── Only ONE foreground job at a time
│
├── Background job 1 (runs independently)
├── Background job 2 (runs independently)
└── Background job 3 (runs independently)
    └── Multiple background jobs allowed
```

### Job States:
| State | Description | How to get here |
|-------|-------------|----------------|
| **Running (fg)** | Executing, blocking terminal | Normal execution |
| **Running (bg)** | Executing in background | `command &` or `bg` |
| **Stopped** | Paused, not executing | `Ctrl+Z` or `kill -STOP` |
| **Done** | Completed | Natural exit |

### The `&` operator:
```bash
$ command &          # Start directly in background
$ command &          # Shell gives you [job_number] PID
[1] 12345            # Job 1, PID 12345
```

### Job numbers vs PIDs:
- **Job number** `%1` — Shell-specific, used with `fg`/`bg`/`jobs`
- **PID** `12345` — System-wide, used with `kill`/`ps`

---

## 🧰 Syntax & Options

### `jobs`:
```bash
jobs [OPTIONS]
```

| Flag | Description |
|------|-------------|
| `-l` | Show PIDs along with job info |
| `-p` | Show only PIDs |
| `-r` | Show only running jobs |
| `-s` | Show only stopped jobs |

### `bg`:
```bash
bg [%JOB_NUMBER]
```

### `fg`:
```bash
fg [%JOB_NUMBER]
```

### Job references:
| Reference | Meaning |
|-----------|---------|
| `%1` | Job number 1 |
| `%+` or `%%` | Current (most recent) job |
| `%-` | Previous job |
| `%string` | Job whose command starts with "string" |
| `%?string` | Job whose command contains "string" |

---

## 🟢 Basic Usage

### Start a background job
```bash
# Method 1: Use & at the end
$ sleep 100 &
[1] 12345

# Method 2: Start normally → Ctrl+Z → bg
$ python3 long_script.py
^Z                            # Ctrl+Z pauses it
[1]+  Stopped    python3 long_script.py
$ bg                          # Resume in background
[1]+ python3 long_script.py &
```

### List running jobs
```bash
$ jobs
[1]+  Running                 sleep 100 &
[2]-  Stopped                 python3 app.py
[3]   Running                 wget https://example.com/big.iso &

# With PIDs
$ jobs -l
[1]+ 12345 Running                 sleep 100 &
[2]- 12346 Stopped                 python3 app.py
```

### Bring to foreground
```bash
$ fg           # Bring most recent job to foreground
$ fg %1        # Bring job 1 to foreground
$ fg %2        # Bring job 2 to foreground
```

### Send to background
```bash
# First pause with Ctrl+Z, then:
$ bg           # Resume most recent in background
$ bg %2        # Resume job 2 in background
```

---

## 🟡 Intermediate Usage

### Multiple background jobs workflow
```bash
# Start multiple background tasks
$ find / -name "*.log" > /tmp/logs.txt 2>/dev/null &
[1] 1001
$ wget https://example.com/dataset.zip &
[2] 1002
$ python3 process_data.py &
[3] 1003

# Check all jobs
$ jobs
[1]   Running    find / -name "*.log" > /tmp/logs.txt 2>/dev/null &
[2]-  Running    wget https://example.com/dataset.zip &
[3]+  Running    python3 process_data.py &

# Bring specific job to foreground
$ fg %2        # Check wget progress

# Ctrl+Z to pause it, then bg to continue downloading
^Z
$ bg %2
```

### Wait for background jobs to finish
```bash
# Wait for ALL background jobs
$ wait

# Wait for specific job
$ wait %1
$ wait 12345       # By PID

# Use in scripts
$ task1 &
$ task2 &
$ task3 &
$ wait             # Script waits until ALL tasks complete
$ echo "All done!"
```

### Redirect background output
```bash
# Background jobs print to terminal — messy! Redirect instead:
$ long_task > output.log 2>&1 &

# Separate stdout and stderr
$ long_task > output.log 2> error.log &

# Discard all output
$ noisy_task > /dev/null 2>&1 &
```

### Disown — Detach from terminal
```bash
# Start a job and detach it from terminal
$ long_task &
$ disown %1        # Job continues even if terminal closes

# Start, background, and disown in one flow
$ long_task &
$ disown

# Disown all jobs
$ disown -a

# Disown only running jobs
$ disown -r
```

---

## 🔴 Advanced Usage

### Parallel Execution Pattern
```bash
#!/bin/bash
# parallel_tasks.sh — Run tasks in parallel, wait for all
MAX_JOBS=4

for file in /data/input/*.csv; do
    # Run in background
    python3 process.py "$file" &
    
    # Limit concurrent jobs
    while [ $(jobs -r | wc -l) -ge $MAX_JOBS ]; do
        sleep 1
    done
done

wait    # Wait for all remaining jobs
echo "All files processed!"
```

### Trap cleanup on exit
```bash
#!/bin/bash
# Clean up background jobs on script exit
cleanup() {
    echo "Cleaning up..."
    kill $(jobs -p) 2>/dev/null
    wait
}
trap cleanup EXIT INT TERM

# Start background workers
worker1 &
worker2 &
worker3 &

wait
```

### Job control in pentesting 🎯
```bash
# Run multiple scans in background
$ nmap -sV -p- target1 > scan1.txt 2>&1 &
$ nmap -sV -p- target2 > scan2.txt 2>&1 &
$ nmap -sV -p- target3 > scan3.txt 2>&1 &

# Check progress
$ jobs
[1]   Running    nmap -sV -p- target1 ...
[2]-  Running    nmap -sV -p- target2 ...
[3]+  Running    nmap -sV -p- target3 ...

# Bring one to foreground to check output
$ fg %1

# Start reverse shell listener in background
$ nc -lvnp 4444 &
# Continue doing other work...
```

### Subshell execution
```bash
# Run command group in background
$ (cd /backup && tar czf backup.tar.gz . && echo "Done!") &

# Pipeline in background
$ (find / -name "*.conf" | xargs grep "password") > results.txt 2>/dev/null &
```

---

## 🔗 Piping & Combining

```bash
# Kill all background jobs
$ kill $(jobs -p)

# Kill all stopped jobs
$ kill $(jobs -ps)

# Count running background jobs
$ jobs -r | wc -l

# Wait and notify
$ long_task &
$ PID=$!              # $! = PID of last background process
$ wait $PID && echo "Task completed!" || echo "Task failed!"
```

---

## 💡 Real World Pro Tips

### Tip 1: `$!` captures last background PID
```bash
$ long_task &
$ echo $!         # PID of long_task
12345

# Use in scripts
$ server &
$ SERVER_PID=$!
# ... later ...
$ kill $SERVER_PID
```

### Tip 2: Use `nohup` for persistent background tasks
```bash
# Jobs die when terminal closes! Use nohup to prevent:
$ nohup long_task > output.log 2>&1 &
$ disown
# Now safe to close terminal
```

### Tip 3: Prefer `tmux`/`screen` over job control
```bash
# For serious multitasking, use tmux:
$ tmux new -s work
# Run tasks, detach with Ctrl+B, D
# Reattach: tmux attach -t work
```

### Tip 4: Check exit status of background job
```bash
$ task &
$ PID=$!
$ wait $PID
$ echo $?     # Exit code of the background task
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Multitask in single terminal | Jobs die when terminal closes |
| Simple `&` syntax | Output mixes with terminal |
| Pause/resume with Ctrl+Z | No persistence (use tmux) |
| Wait for completion | Limited to current shell session |
| Parallel execution | No built-in max concurrency |

---

## 📍 Where & When to Use

| Scenario | Use | Why |
|----------|-----|-----|
| Quick background task | `command &` | Simple one-off |
| Pause current task | `Ctrl+Z` | Need terminal temporarily |
| Multiple scans running | `&` + `jobs` | Parallel pentesting |
| Script parallelism | `& + wait` | Faster execution |
| Persistent background | `nohup` or `tmux` | Survives terminal close |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Background job output clutters terminal | Redirect: `cmd > file 2>&1 &` |
| Closing terminal kills background jobs | Use `nohup` or `disown` |
| Forgetting `&` at end of command | Add `&` or use `Ctrl+Z` + `bg` |
| Not checking if jobs finished | Use `jobs` or `wait` |
| Too many background jobs | Limit with job counter loop |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Start `sleep 60 &` and check with `jobs`
2. Run a command, pause with `Ctrl+Z`, resume with `bg`
3. Bring a background job to foreground with `fg`

### 🟡 Intermediate
4. Run 3 background tasks and wait for all
5. Use `disown` to detach a job from terminal
6. Redirect background job output to a file

### 🔴 Advanced
7. Write a parallel processing script with max job limit
8. Create a trap handler to clean up background jobs
9. Run multiple nmap scans in background and monitor

---

## 🧠 Cheat Sheet

```
START BACKGROUND:
  command &              → Start in background
  Ctrl+Z → bg           → Pause then background

JOB CONTROL:
  jobs                   → List all jobs
  jobs -l                → List with PIDs
  fg %N                  → Foreground job N
  bg %N                  → Background job N
  fg                     → Foreground latest job

KEYBOARD:
  Ctrl+C                 → Kill foreground
  Ctrl+Z                 → Pause foreground

DETACH:
  disown %N              → Detach job N
  nohup cmd &            → Survive terminal close

SCRIPTING:
  $!                     → PID of last background job
  wait                   → Wait for all jobs
  wait $PID              → Wait for specific job
  kill $(jobs -p)        → Kill all jobs
```

---

> **Previous**: [`kill/killall` ←](./29_kill_killall.md) | **Next**: [`nohup` →](./31_nohup.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
