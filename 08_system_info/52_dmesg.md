# 🛠️ `dmesg` — Kernel Ring Buffer Messages | Linux Master Note

> **See what the kernel is thinking. `dmesg` shows hardware events, driver loading, boot messages, and system errors — your window into the kernel's soul.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--kernel-ring-buffer)
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

### What is `dmesg`?
`dmesg` (**D**iagnostic **Mes**sa**g**e) displays the **kernel ring buffer** — a circular log where the kernel records hardware events, driver messages, boot info, and errors.

### Why it matters:
- **Hardware troubleshooting** — Disk failures, USB detection, network errors
- **Boot analysis** — What happened during system startup
- **Driver debugging** — Module loading/unloading issues
- **Security** — Detect suspicious hardware, keyloggers, rogue USB devices
- **Forensics** — Analyze hardware events timeline

---

## 📖 Theory — Kernel Ring Buffer

### What is the ring buffer?
```
Kernel starts → Messages fill buffer → Older messages overwritten
┌──────────────────────────────────────┐
│ [boot] [driver] [usb] [net] [disk]  │ ← Circular buffer
│  ↑ oldest                  newest ↑  │
└──────────────────────────────────────┘
Default size: 64KB - 512KB (configurable)
```

### Log levels (severity):
| Level | Name | Value | Description |
|-------|------|-------|-------------|
| 0 | `emerg` | KERN_EMERG | System is unusable |
| 1 | `alert` | KERN_ALERT | Action must be taken immediately |
| 2 | `crit` | KERN_CRIT | Critical conditions |
| 3 | `err` | KERN_ERR | Error conditions |
| 4 | `warn` | KERN_WARNING | Warning conditions |
| 5 | `notice` | KERN_NOTICE | Normal but significant |
| 6 | `info` | KERN_INFO | Informational |
| 7 | `debug` | KERN_DEBUG | Debug-level messages |

### Where dmesg data comes from:
```
Kernel → /dev/kmsg → dmesg reads this
                   → journalctl -k (systemd alternative)
                   → /var/log/kern.log (syslog writes here)
```

---

## 🧰 Syntax & Options

```bash
dmesg [OPTIONS]
```

| Flag | Description |
|------|-------------|
| `-T` | **Human-readable timestamps** (MOST USEFUL!) |
| `-H` | Human-readable format (pager + color) |
| `-w` | **Follow** (like tail -f) |
| `-l LEVEL` | Filter by log **level** |
| `-f FACILITY` | Filter by **facility** (kern, user, daemon) |
| `-c` | Clear buffer after reading |
| `-C` | Clear buffer without reading |
| `-n LEVEL` | Set console log level |
| `--color=always` | Force color output |
| `-x` | Decode facility and level |
| `-k` | Kernel messages only |
| `-u` | Userspace messages only |
| `-t` | Don't print timestamps |
| `-r` | Raw format |

---

## 🟢 Basic Usage

```bash
# View all kernel messages
$ dmesg

# Human-readable timestamps (THE way to use dmesg!)
$ sudo dmesg -T
[Mon May 26 01:00:00 2026] Linux version 5.15.0-91-generic
[Mon May 26 01:00:01 2026] Command line: BOOT_IMAGE=/vmlinuz
[Mon May 26 01:00:01 2026] Memory: 16384MB available

# Human-readable format with pager + colors
$ sudo dmesg -H

# Follow new messages (like tail -f)
$ sudo dmesg -w
# Now plug in USB, watch message appear!

# Last 20 messages
$ dmesg | tail -20

# Clear buffer (after reading)
$ sudo dmesg -c > /tmp/dmesg_backup.txt
```

---

## 🟡 Intermediate Usage

### Filter by log level
```bash
# Only errors
$ dmesg -l err
$ dmesg -l err,crit,alert,emerg    # All serious issues

# Only warnings
$ dmesg -l warn

# Errors + warnings
$ dmesg -l err,warn

# Info messages
$ dmesg -l info
```

### Filter by content
```bash
# USB events
$ dmesg | grep -i usb
[  2.345] usb 1-1: new high-speed USB device
[  2.456] usb 1-1: Product: USB Flash Drive
[  2.567] usb-storage 1-1: USB Mass Storage device detected

# Disk/storage events
$ dmesg | grep -iE "sd[a-z]|nvme|disk|ata"
[  1.234] ata1: SATA max UDMA/133
[  1.345] sd 0:0:0:0: [sda] 500107862016 512-byte sectors

# Network events
$ dmesg | grep -iE "eth|wlan|enp|wlp|link"
[  3.456] e1000: enp0s3 NIC Link is Up 1000 Mbps

# Memory events
$ dmesg | grep -iE "memory|oom|swap"

# CPU events
$ dmesg | grep -iE "cpu|processor|microcode"
```

### Decode facility and level
```bash
$ dmesg -x | head -5
kern  :info  : [    0.000000] Linux version 5.15.0-91-generic
kern  :info  : [    0.000000] Command line: BOOT_IMAGE=/vmlinuz
kern  :notice: [    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff]
```

### Modern alternative: journalctl
```bash
# Same as dmesg but with more features
$ journalctl -k                    # Kernel messages (current boot)
$ journalctl -k -b -1             # Previous boot
$ journalctl -k -p err            # Only errors
$ journalctl -k --since "1 hour ago"
$ journalctl -k -f                # Follow
```

---

## 🔴 Advanced Usage

### Hardware Troubleshooting
```bash
# Disk errors (CRITICAL — data loss risk!)
$ dmesg -T | grep -iE "error|fail|i/o|bad|sector|reset"
[May 26 10:30:15] ata1.00: failed command: READ FPDMA QUEUED
[May 26 10:30:15] ata1.00: status: { DRDY ERR }
[May 26 10:30:15] ata1.00: error: { UNC }
# ⚠️ UNC = Uncorrectable error → disk is DYING!

# Memory errors
$ dmesg -T | grep -iE "mce|memory|ecc|edac"

# GPU issues
$ dmesg -T | grep -iE "drm|gpu|nvidia|amdgpu|radeon"
```

### Boot analysis
```bash
# Full boot sequence with timestamps
$ dmesg -T | head -100

# Boot time analysis
$ systemd-analyze                  # Total boot time
$ systemd-analyze blame            # Slowest services
$ dmesg -T | grep "Link is Up"    # When network came up
```

### Security — USB/Hardware Monitoring 🔒
```bash
# Detect USB device insertions
$ dmesg -Tw | grep -i usb
# Plug in a device:
[May 26 01:15:30] usb 1-2: new high-speed USB device number 4
[May 26 01:15:30] usb 1-2: Product: BadUSB_Keylogger  # ⚠️ SUSPICIOUS!

# Monitor for new storage devices (data exfiltration?)
$ dmesg -Tw | grep -iE "sd[a-z]|usb-storage"

# Check for hardware keyloggers
$ dmesg | grep -i "keyboard\|hid\|input"

# Detect suspicious kernel modules loaded
$ dmesg | grep -iE "module|insmod|modprobe"

# Script: USB watchdog
#!/bin/bash
dmesg -Tw | while read line; do
    if echo "$line" | grep -qi "usb.*new"; then
        echo "⚠️ USB DEVICE DETECTED: $line" | tee -a /var/log/usb_monitor.log
    fi
done
```

### Forensics — Event Timeline 🕵️
```bash
# Capture full dmesg for evidence
$ sudo dmesg -T > /evidence/dmesg_$(date +%Y%m%d_%H%M%S).txt

# Find OOM (Out of Memory) kills
$ dmesg -T | grep -i "oom\|out of memory\|killed process"
[May 25 23:45:00] Out of memory: Killed process 1234 (mysql)
# → Someone or something consumed all RAM!

# Detect kernel panic history
$ dmesg | grep -i "panic\|oops\|bug\|call trace"
```

---

## 💡 Real World Pro Tips

### Tip 1: Always use `-T` for readable timestamps
```bash
$ sudo dmesg -T | tail -20
# Without -T you get seconds since boot (useless for humans)
```

### Tip 2: Live monitor hardware changes
```bash
$ sudo dmesg -Tw
# Now plug/unplug USB, attach disk, connect network
```

### Tip 3: Check after strange system behavior
```bash
# System acting weird? Check dmesg first!
$ dmesg -T -l err,crit,alert,emerg | tail -30
```

### Tip 4: Non-root access
```bash
# If dmesg requires root (kernel.dmesg_restrict=1):
$ sudo sysctl kernel.dmesg_restrict=0    # Temporary allow
# Or use journalctl -k (respects journal permissions)
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Direct kernel access | Ring buffer overwrites old messages |
| Fast, always available | Timestamps relative (without -T) |
| Hardware event logging | May require root |
| Live following (-w) | No persistent history (use journalctl) |
| Log level filtering | Can be noisy |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| USB not detected | `dmesg -T \| grep usb` | Driver issues |
| Disk errors | `dmesg -T \| grep -i error` | Hardware failure |
| Network issues | `dmesg -T \| grep -i eth` | Link status |
| System crash | `dmesg -T -l err,crit` | Find errors |
| Hardware monitor | `dmesg -Tw` | Live events |
| Forensics | `dmesg -T > evidence.txt` | Event capture |
| Boot analysis | `dmesg -T \| head -100` | Startup sequence |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Not using `-T` (unreadable timestamps) | Always `dmesg -T` |
| Forgetting sudo | `sudo dmesg -T` |
| Not checking after hardware issues | `dmesg` should be your first stop |
| Ignoring disk errors in dmesg | UNC/I/O errors = disk dying |
| Not capturing before reboot | Save with `dmesg -T > file` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. View kernel messages with `dmesg -T`
2. Find USB-related messages
3. Show only error-level messages

### 🟡 Intermediate
4. Monitor dmesg live while plugging in USB
5. Find disk and network events
6. Compare `dmesg -k` with `journalctl -k`

### 🔴 Advanced
7. Write a USB watchdog monitoring script
8. Analyze boot sequence for performance
9. Capture dmesg for forensic evidence preservation

---

## 🧠 Cheat Sheet

```
dmesg -T                    → Human timestamps (USE THIS!)
dmesg -H                    → Pretty format with pager
dmesg -Tw                   → Follow live (like tail -f)
dmesg -l err                → Only errors
dmesg -l err,crit,alert     → Serious issues only
dmesg -x                    → Show facility + level

FILTER:
  dmesg -T | grep -i usb    → USB events
  dmesg -T | grep -i error  → Errors
  dmesg -T | grep -iE "sd[a-z]"  → Disk events
  dmesg -T | grep -i eth    → Network events
  dmesg -T | grep -i oom    → Out of memory

MODERN ALTERNATIVE:
  journalctl -k              → Kernel messages
  journalctl -k -b -1        → Previous boot
  journalctl -k -p err       → Only errors
  journalctl -k -f           → Follow live

FORENSICS:
  sudo dmesg -T > evidence.txt  → Capture for analysis
  sudo dmesg -c                  → Read and clear
```

---

> **Previous**: [`hostname` ←](./51_hostname.md) | **Next**: [`uptime` →](./53_uptime.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
