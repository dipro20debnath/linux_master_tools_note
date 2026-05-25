# 🛠️ `free` — Memory Usage Display | Linux Master Note

> **How much RAM is left? `free` shows physical memory, swap, and the all-important 'available' column — your instant memory health check.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--linux-memory-model)
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

### What is `free`?
`free` displays the amount of **used and available** physical memory (RAM) and swap space. It reads from `/proc/meminfo`.

### Why it matters:
- **Server health** — RAM exhaustion = OOM kills = service crashes
- **Performance tuning** — Identify memory bottlenecks
- **Capacity planning** — Know when to upgrade RAM
- **Security** — Detect cryptominers and memory-hungry malware

---

## 📖 Theory — Linux Memory Model

### Memory columns explained:
```
              total       used       free     shared  buff/cache   available
Mem:          16Gi       5.2Gi      1.8Gi     256Mi      9.0Gi      10.2Gi
Swap:          4Gi       0.0Gi      4.0Gi
```

| Column | Meaning |
|--------|---------|
| `total` | Total physical RAM |
| `used` | RAM actively in use |
| `free` | Completely unused RAM |
| `shared` | Memory used by tmpfs |
| `buff/cache` | **Buffers + cache (reclaimable!)** |
| `available` | **🔥 ACTUALLY available for new apps** |

### The MOST important insight:
```
❌ DON'T PANIC: "Only 1.8GB free out of 16GB!"
✅ LOOK AT:    "10.2GB available" ← This is what matters!

Linux uses "free" RAM for disk cache (buff/cache)
This cache is AUTOMATICALLY released when apps need memory
The "available" column = free + reclaimable cache
```

### Buffers vs Cache:
| Type | What it caches |
|------|---------------|
| **Buffers** | Raw disk block data (metadata, directory entries) |
| **Cache** | File content (page cache — speeds up reads) |

### What happens when RAM runs out:
```
RAM nearly full → Kernel starts swapping → Performance degrades
                → Still not enough → OOM Killer activates!
                → Kills largest/lowest priority process
                → Check: dmesg | grep -i "oom\|killed process"
```

---

## 🧰 Syntax & Options

```bash
free [OPTIONS]
```

| Flag | Description |
|------|-------------|
| `-h` | **Human-readable** (KiB, MiB, GiB) — MOST USED |
| `-b` | Bytes |
| `-k` | Kibibytes (default) |
| `-m` | Mebibytes |
| `-g` | Gibibytes |
| `--tera` | Tebibytes |
| `-s N` | **Repeat** every N seconds |
| `-c N` | Repeat N times (with `-s`) |
| `-t` | Show **total** row |
| `-w` | **Wide** output (separate buffers/cache) |
| `-l` | Show low/high memory stats |
| `--si` | Use SI units (1000 instead of 1024) |

---

## 🟢 Basic Usage

```bash
# Human-readable (THE command to use)
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           16Gi       5.2Gi       1.8Gi       256Mi       9.0Gi       10.2Gi
Swap:          4.0Gi          0B       4.0Gi

# In megabytes
$ free -m

# Wide mode (separates buffers from cache)
$ free -wh
              total        used        free      shared     buffers       cache   available
Mem:           16Gi       5.2Gi       1.8Gi       256Mi       512Mi       8.5Gi       10.2Gi

# With total row
$ free -ht
```

---

## 🟡 Intermediate Usage

### Continuous monitoring
```bash
# Update every 2 seconds
$ free -h -s 2

# Update every 5 seconds, 10 times
$ free -h -s 5 -c 10

# With watch (better visual)
$ watch -n 2 free -h
```

### Detailed memory from /proc/meminfo
```bash
$ cat /proc/meminfo
MemTotal:       16384000 kB
MemFree:         1843200 kB
MemAvailable:   10444800 kB
Buffers:          524288 kB
Cached:          8704000 kB
SwapCached:            0 kB
SwapTotal:       4194304 kB
SwapFree:        4194304 kB
Dirty:               128 kB
Slab:             819200 kB
SReclaimable:     614400 kB
SUnreclaim:       204800 kB

# Quick memory summary
$ grep -E "^(MemTotal|MemFree|MemAvailable|Buffers|Cached|SwapTotal|SwapFree)" /proc/meminfo
```

### Memory percentage calculation
```bash
# Get memory usage percentage
$ free | awk '/Mem:/ {printf "RAM: %.1f%% used\n", ($3/$2)*100}'
RAM: 31.7% used

# Available percentage
$ free | awk '/Mem:/ {printf "Available: %.1f%%\n", ($7/$2)*100}'
Available: 62.5%
```

### Swap analysis
```bash
# Check swap usage
$ free -h | grep Swap
Swap:          4.0Gi       512Mi       3.5Gi

# If swap is being used heavily → system is memory-constrained!
# Check what's using swap:
$ for pid in /proc/[0-9]*; do
    name=$(cat "$pid/comm" 2>/dev/null)
    swap=$(grep VmSwap "$pid/status" 2>/dev/null | awk '{print $2}')
    [ -n "$swap" ] && [ "$swap" -gt 0 ] && echo "$swap kB - $name (PID $(basename $pid))"
done | sort -rn | head -10
```

---

## 🔴 Advanced Usage

### Memory Monitoring Script
```bash
#!/bin/bash
# mem_monitor.sh — Alert when available memory is low
THRESHOLD=15  # Alert if available < 15%

while true; do
    AVAIL_PCT=$(free | awk '/Mem:/ {printf "%.0f", ($7/$2)*100}')
    
    if [ "$AVAIL_PCT" -lt "$THRESHOLD" ]; then
        echo "⚠️ LOW MEMORY! Available: ${AVAIL_PCT}% (threshold: ${THRESHOLD}%)"
        echo "Top memory consumers:"
        ps aux --sort=-%mem | head -6
        echo ""
    fi
    
    sleep 30
done
```

### Drop caches (free up buff/cache)
```bash
# Free page cache
$ sudo sync; echo 1 | sudo tee /proc/sys/vm/drop_caches

# Free dentries and inodes
$ sudo sync; echo 2 | sudo tee /proc/sys/vm/drop_caches

# Free all (page cache + dentries + inodes)
$ sudo sync; echo 3 | sudo tee /proc/sys/vm/drop_caches

# ⚠️ Usually NOT needed — Linux manages this automatically!
# Only do this for benchmarking or testing
```

### OOM Killer analysis
```bash
# Check if OOM killer has been active
$ dmesg -T | grep -i "oom\|out of memory\|killed process"
[May 25 23:45:00] Out of memory: Killed process 1234 (mysql) total-vm:8388608kB

# Check OOM score (higher = more likely to be killed)
$ cat /proc/$(pgrep mysql)/oom_score

# Protect critical processes from OOM killer
$ echo -1000 | sudo tee /proc/$(pgrep sshd)/oom_score_adj
```

### Security — Detect Anomalous Memory Usage 🔒
```bash
# Find top memory consumers (cryptominer detection!)
$ ps aux --sort=-%mem | head -10
USER       PID %CPU %MEM    VSZ   RSS TTY COMMAND
mysql     1234  2.0 25.0 8388608 4194304 ? Ssl /usr/sbin/mysqld
www-data  5678 95.0 40.0 6291456 6553600 ? R   ./xmrig  # ⚠️ CRYPTOMINER!

# Memory usage by user
$ ps aux | awk '{arr[$1]+=$4} END {for (user in arr) printf "%s: %.1f%%\n", user, arr[user]}' | sort -t: -k2 -rn

# System memory report for forensics
#!/bin/bash
echo "=== MEMORY FORENSICS — $(date) ==="
echo ""
free -h
echo ""
echo "=== TOP MEMORY PROCESSES ==="
ps aux --sort=-%mem | head -15
echo ""
echo "=== SWAP USAGE BY PROCESS ==="
for pid in /proc/[0-9]*; do
    name=$(cat "$pid/comm" 2>/dev/null)
    swap=$(grep VmSwap "$pid/status" 2>/dev/null | awk '{print $2}')
    [ -n "$swap" ] && [ "$swap" -gt 0 ] && echo "${swap}kB - $name"
done | sort -rn | head -10
```

---

## 💡 Real World Pro Tips

### Tip 1: LOOK AT 'available', NOT 'free'!
```bash
# ❌ Wrong: "Only 1.8GB free! Server is out of RAM!"
# ✅ Right: "10.2GB available — that's fine."
$ free -h | awk '/Mem:/ {print "Available:", $7}'
```

### Tip 2: Swap usage means you need more RAM
```bash
# If swap used > 0, system is memory-constrained
$ free -h | awk '/Swap:/ {print "Swap used:", $3}'
```

### Tip 3: Quick one-liner check
```bash
$ free -h | awk '/Mem:/ {printf "RAM: %s/%s (Available: %s)\n", $3, $2, $7}'
RAM: 5.2Gi/16Gi (Available: 10.2Gi)
```

### Tip 4: vmstat for detailed view
```bash
$ vmstat 1 5    # Every 1 second, 5 times
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 1843200 524288 8704000    0    0    12     8  250  500 15  3 80  2  0
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Instant memory overview | No per-process breakdown |
| Human-readable output | 'used' column misleading to beginners |
| Continuous monitoring (-s) | No historical data |
| Shows swap usage | Doesn't show OOM risk level |
| Simple and fast | No graphical output |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Quick RAM check | `free -h` | Instant overview |
| Continuous monitor | `free -h -s 2` | Watch memory |
| Server health | Check 'available' column | Actual availability |
| Swap check | `free -h \| grep Swap` | Memory pressure |
| Memory report | Script with `free` + `ps` | Full analysis |
| Drop cache (testing) | `echo 3 > drop_caches` | Benchmarking |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Panicking at low 'free' | Look at **'available'** instead! |
| Thinking buff/cache is wasted | It's **reclaimable** — this is GOOD |
| Not checking swap usage | Swap > 0 = memory pressure |
| `sudo pip install` eating RAM | Use venv (wrong tool, same issue) |
| Not monitoring before crashes | Set up memory alerts |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Check memory with `free -h` and identify each column
2. Monitor memory continuously with `free -h -s 2`
3. Use wide mode with `free -wh`

### 🟡 Intermediate
4. Calculate memory usage percentage with awk
5. Examine `/proc/meminfo` for detailed stats
6. Find top memory-consuming processes

### 🔴 Advanced
7. Write a memory monitoring script with alerts
8. Investigate OOM killer events in dmesg
9. Find processes using swap and diagnose memory pressure

---

## 🧠 Cheat Sheet

```
free -h                  → Human-readable (USE THIS!)
free -wh                 → Wide (separate buffers/cache)
free -ht                 → With total row
free -h -s 2             → Update every 2 seconds
free -h -s 5 -c 10       → 10 updates, 5s apart

KEY INSIGHT:
  'available' = what you can ACTUALLY use
  'free' = unused (buff/cache is also usable!)
  buff/cache = disk cache (auto-released when needed)

MONITORING:
  watch -n 2 free -h                         → Visual monitor
  free | awk '/Mem:/ {printf "%.0f%%\n", ($7/$2)*100}'  → Available %

TROUBLESHOOTING:
  free -h | grep Swap             → Check swap usage
  ps aux --sort=-%mem | head      → Top RAM consumers
  dmesg -T | grep -i oom          → OOM kills
  cat /proc/meminfo               → Full details
  vmstat 1 5                      → Detailed stats

DROP CACHE (testing only):
  sudo sync; echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

> **Previous**: [`uptime` ←](./53_uptime.md) | **Next**: [`lscpu/lshw` →](./55_lscpu_lshw.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
