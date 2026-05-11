# 🛠️ `top` & `htop` — Real-Time Process Monitoring | Linux Master Note

> **Your live dashboard into the system. `top` is the heartbeat monitor of Linux — see CPU, memory, and every process in real-time.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--system-metrics)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [htop — The Better top](#-htop--the-better-top)
8. [Real World Pro Tips](#-real-world-pro-tips)
9. [Pros & Cons](#-pros--cons)
10. [Where & When to Use](#-where--when-to-use)
11. [Common Mistakes](#-common-mistakes)
12. [Practice Exercises](#-practice-exercises)
13. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### `top` vs `htop`:
| Feature | `top` | `htop` |
|---------|-------|--------|
| Pre-installed | ✅ Always available | ❌ Must install |
| Interface | Text-based | Colorful, visual |
| Mouse support | ❌ No | ✅ Yes |
| Scroll | ❌ No | ✅ Yes |
| Process tree | ❌ Limited | ✅ Built-in |
| Kill process | Via PID typing | Click/select + F9 |
| CPU per core | ❌ No (default) | ✅ Visual bars |

### Why it matters:
- **Performance debugging** — Find what's slowing your system
- **Memory leaks** — Watch memory grow over time
- **Server monitoring** — Is the server overloaded?
- **Incident response** — Cryptominer eating CPU? You'll see it here

---

## 📖 Theory — System Metrics

### Load Average (most misunderstood metric!):
```
load average: 0.52, 0.78, 0.95
               │      │      │
               1 min  5 min  15 min
```
- **Load = number of processes waiting for CPU**
- Load 1.0 on 1-core = 100% utilized
- Load 4.0 on 4-core = 100% utilized
- Load > number of cores = **overloaded!**

### CPU States:
| Code | Meaning | Normal Range |
|------|---------|-------------|
| `us` | **User** — your programs | 0-70% |
| `sy` | **System** — kernel work | 0-30% |
| `ni` | **Nice** — low-priority processes | 0-20% |
| `id` | **Idle** — doing nothing | 30-99% |
| `wa` | **I/O Wait** — waiting for disk | 0-5% ⚠️ |
| `hi` | Hardware interrupts | 0-5% |
| `si` | Software interrupts | 0-5% |
| `st` | **Steal** — VM hypervisor stole CPU | 0-5% ⚠️ |

> 🔴 High `wa` = disk bottleneck. High `st` = VM resource contention.

### Memory fields:
| Field | Meaning |
|-------|---------|
| `VIRT` (VSZ) | Total virtual memory allocated |
| `RES` (RSS) | **Actual physical RAM used** (this matters!) |
| `SHR` | Shared memory (libraries, etc.) |
| `%MEM` | Percentage of total RAM |

---

## 🧰 Syntax & Options

### `top` command-line options:
```bash
top [OPTIONS]
```

| Flag | Description |
|------|-------------|
| `-d SEC` | Update interval (default: 3 seconds) |
| `-n NUM` | Number of iterations then exit |
| `-u USER` | Show only user's processes |
| `-p PID` | Monitor specific PID(s) |
| `-b` | Batch mode (for scripting/logging) |
| `-c` | Show full command line |
| `-H` | Show individual threads |
| `-i` | Don't show idle processes |
| `-o FIELD` | Sort by field |

### Interactive keys (while top is running):

| Key | Action |
|-----|--------|
| `q` | Quit |
| `h` or `?` | Help |
| `P` | Sort by **CPU** |
| `M` | Sort by **Memory** |
| `T` | Sort by **Time** |
| `N` | Sort by **PID** |
| `k` | Kill a process (enter PID) |
| `r` | Renice a process |
| `c` | Toggle full command path |
| `1` | Toggle per-CPU core display |
| `t` | Toggle CPU graph display |
| `m` | Toggle memory graph display |
| `f` | Add/remove display fields |
| `u` | Filter by user |
| `W` | Save current config |
| `d` | Change refresh interval |
| `Space` | Force refresh |

---

## 🟢 Basic Usage

```bash
# Launch top (real-time view)
$ top

# Top output explained:
top - 14:30:52 up 3 days, 2:15,  2 users,  load average: 0.52, 0.78, 0.95
Tasks: 245 total,   1 running, 244 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.2 us,  2.1 sy,  0.0 ni, 92.3 id,  0.3 wa,  0.0 hi,  0.1 si,  0.0 st
MiB Mem :  16384.0 total,   4096.0 free,   8192.0 used,   4096.0 buff/cache
MiB Swap:   4096.0 total,   4096.0 free,      0.0 used.   7680.0 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 5678 dipro     20   0 2456780 987600  45600 S  45.2   6.0   5:23.45 python3
 1234 mysql     20   0  856420 458760  12340 S   2.1   2.8   1:15.67 mysqld

# Monitor specific user
$ top -u dipro

# Monitor specific PID
$ top -p 1234

# Fast refresh (every 1 second)
$ top -d 1
```

### While inside `top`:
```
Press P → Sort by CPU (find CPU hog)
Press M → Sort by Memory (find memory hog)
Press 1 → Show per-CPU core usage
Press c → Show full command path
Press k → Enter PID to kill a process
```

---

## 🟡 Intermediate Usage

### Batch mode (for logging/scripting)
```bash
# Capture 5 snapshots, 2 seconds apart
$ top -b -n 5 -d 2 > system_report.txt

# Get top CPU consumers (one-shot)
$ top -b -n 1 | head -20

# Filter specific user in batch mode
$ top -b -n 1 -u www-data
```

### Custom display fields
```bash
# Press 'f' inside top to add/remove fields:
# Useful fields to add:
# - PPID (parent PID)
# - nTH (thread count)  
# - ENVIRON (environment variables)
# - SWAP (swap usage)
```

### Per-core CPU monitoring
```bash
# Press '1' inside top:
%Cpu0  : 25.0 us,  5.0 sy,  0.0 ni, 70.0 id
%Cpu1  :  3.0 us,  1.0 sy,  0.0 ni, 96.0 id
%Cpu2  : 80.0 us,  5.0 sy,  0.0 ni, 15.0 id    # ← This core is busy!
%Cpu3  :  1.0 us,  0.5 sy,  0.0 ni, 98.5 id
```

### Kill process from top
```bash
# Press 'k' inside top
# Enter PID: 5678
# Enter signal (default 15 = SIGTERM, use 9 = SIGKILL for force)
```

### Renice process from top
```bash
# Press 'r' inside top
# Enter PID: 5678
# Enter nice value: 10 (lower priority)
```

---

## 🔴 Advanced Usage

### Security Monitoring 🔒
```bash
# Batch mode — capture for forensics
$ top -b -n 1 -c | grep -E '/tmp/|/dev/shm/|miner|crypto'

# Watch for sudden CPU spikes (cryptominer detection)
$ top -b -d 5 | awk '/^%Cpu/ {if ($2 > 90) print "HIGH CPU:", $0}'

# Find process consuming most CPU over time
$ top -b -n 60 -d 1 | grep "^ *[0-9]" | awk '{cpu[$NF]+=$9; count[$NF]++} END {for (p in cpu) printf "%s: avg %.1f%%\n", p, cpu[p]/count[p]}' | sort -t: -k2 -rn | head -10
```

### Performance Analysis Script
```bash
#!/bin/bash
# system_health.sh — Quick system health check
echo "=== SYSTEM HEALTH CHECK ==="
echo "Load Average: $(cat /proc/loadavg | awk '{print $1, $2, $3}')"
echo "CPU Cores: $(nproc)"
echo ""
echo "=== TOP 5 CPU PROCESSES ==="
ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu --no-headers | head -5
echo ""
echo "=== TOP 5 MEMORY PROCESSES ==="
ps -eo pid,user,%cpu,%mem,cmd --sort=-%mem --no-headers | head -5
echo ""
echo "=== MEMORY USAGE ==="
free -h | head -2
echo ""
echo "=== ZOMBIES ==="
ps aux | awk '$8 ~ /Z/ {print $2, $11}' || echo "None"
```

---

## 🌈 htop — The Better top

### Install
```bash
$ sudo apt install htop       # Debian/Ubuntu
$ sudo yum install htop       # RHEL/CentOS
$ sudo pacman -S htop         # Arch
```

### Launch
```bash
$ htop
```

### htop Interactive Keys:

| Key | Action |
|-----|--------|
| `F1` | Help |
| `F2` | Setup (customize display) |
| `F3` | Search process |
| `F4` | Filter processes |
| `F5` | Tree view toggle |
| `F6` | Sort by column |
| `F7` | Decrease nice (higher priority) |
| `F8` | Increase nice (lower priority) |
| `F9` | Kill process (signal menu) |
| `F10` | Quit |
| `t` | Tree view |
| `H` | Toggle user threads |
| `K` | Toggle kernel threads |
| `p` | Toggle full path |
| `Space` | Tag/untag process |
| `U` | Untag all |
| `u` | Filter by user |
| `/` | Search |
| `\` | Filter |

### htop advantages:
```
┌──────────────────────────────────────────────────────────┐
│ CPU[||||||||||||||||          38.5%]  Tasks: 245, 1 thr  │
│ CPU[||                        5.2%]  Load: 0.52 0.78    │
│ CPU[|||||||||||||||||||||    78.0%]  Uptime: 3 days     │
│ CPU[|||                      10.1%]                      │
│ Mem[||||||||||||||      8.0G/16.0G]                      │
│ Swp[                     0K/4.0G]                        │
├──────────────────────────────────────────────────────────┤
│  PID USER    PRI  NI  VIRT   RES   SHR S CPU% MEM% CMD  │
│ 5678 dipro    20   0 2456M  987M  45M  S 45.2  6.0 py.. │
│ 1234 mysql    20   0  856M  458M  12M  S  2.1  2.8 my.. │
└──────────────────────────────────────────────────────────┘
```

### htop features top doesn't have:
1. **Visual CPU bars** per core
2. **Mouse support** — click to sort, select, kill
3. **Horizontal/vertical scrolling**
4. **Process tree** with collapsible branches
5. **Search and filter** live
6. **Tag multiple processes** and kill at once
7. **Color-coded** resource usage

---

## 💡 Real World Pro Tips

### Tip 1: Save top config
```bash
# Configure top the way you like → Press 'W' to save
# Config saved to ~/.toprc
```

### Tip 2: Monitor specific processes
```bash
$ top -p $(pgrep -d',' nginx)     # Monitor all nginx PIDs
$ htop -p $(pgrep -d',' python3)  # Same for htop
```

### Tip 3: High `wa` (I/O Wait) diagnosis
```bash
# If wa > 10% → disk is the bottleneck
# Find which process is doing I/O:
$ sudo iotop -o        # Shows processes doing I/O
```

### Tip 4: Alternatives
```bash
$ btop          # Modern, beautiful (install separately)
$ glances       # System monitoring (Python-based)
$ nmon          # AIX/Linux performance monitor
```

---

## ✅ Pros & Cons

### `top`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Pre-installed everywhere | UI is old-school |
| Lightweight | No mouse support |
| Batch mode for scripting | Hard to scroll |
| Stable and reliable | Limited customization |

### `htop`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Beautiful visual display | Must install separately |
| Mouse support | Slightly more resource usage |
| Process tree built-in | Config file format differences |
| Easy kill/renice | Not on minimal installations |

---

## 📍 Where & When to Use

| Scenario | Tool | Why |
|----------|------|-----|
| Quick system check | `top` | Always available |
| Daily monitoring | `htop` | Better UX |
| Scripting/logging | `top -b` | Machine-readable output |
| Find CPU hog | `top` → press `P` | Sort by CPU |
| Find memory leak | `htop` | Visual memory bars |
| Server without GUI | `top` | No dependencies |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Confusing VIRT with actual RAM | Look at RES (RSS) for real usage |
| Panic at high memory "used" | Linux uses free RAM for cache — check "available" |
| Not knowing load average meaning | Compare with `nproc` (CPU count) |
| Killing wrong process in top | Double-check PID before pressing Enter |
| Ignoring I/O wait | High `wa` = disk bottleneck, not CPU |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Run `top` and identify the top CPU consumer
2. Press `1` to see per-core CPU usage
3. Install and run `htop`

### 🟡 Intermediate
4. Use batch mode to capture 10 snapshots to a file
5. Monitor only nginx processes with `top -p`
6. Use htop tree view to understand process hierarchy

### 🔴 Advanced
7. Write a script that alerts when CPU > 90%
8. Use `top -b` output to create a performance report
9. Diagnose a high I/O wait situation

---

## 🧠 Cheat Sheet

```
TOP:
  top                    → Launch real-time monitor
  top -d 1               → 1-second refresh
  top -u user            → Filter by user
  top -p PID             → Monitor specific PID
  top -b -n 5            → Batch: 5 snapshots

INSIDE TOP:
  P → Sort CPU    M → Sort Memory    T → Sort Time
  1 → Per-core    c → Full command   k → Kill process
  r → Renice      u → Filter user    W → Save config

HTOP:
  htop                   → Launch htop
  F5 → Tree view    F9 → Kill    F3 → Search
  F4 → Filter       F6 → Sort    F2 → Setup

KEY METRICS:
  Load avg > nproc          → Overloaded!
  wa > 10%                  → Disk bottleneck
  st > 5%                   → VM resource contention
  RES = actual RAM used     → Not VIRT!
```

---

> **Previous**: [`ps` ←](./27_ps.md) | **Next**: [`kill/killall` →](./29_kill_killall.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
