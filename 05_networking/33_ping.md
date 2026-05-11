# 🛠️ `ping` — Test Network Connectivity | Linux Master Note

> **The first command you run when "the internet is down." `ping` sends ICMP echo requests to test if a host is alive and measures round-trip time.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--icmp-protocol)
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

### What is `ping`?
`ping` sends **ICMP Echo Request** packets to a target host and waits for **ICMP Echo Reply**. It measures **latency** (round-trip time) and detects **packet loss**.

### Named after sonar:
> Like a submarine's sonar "ping" — send a sound, listen for the echo, measure the distance.

### Why it matters:
- **Connectivity check** — Is the host alive?
- **Latency measurement** — How fast is the connection?
- **Packet loss** — Is the network reliable?
- **DNS resolution** — Does the hostname resolve?
- **Routing issues** — Where is the connection failing?

---

## 📖 Theory — ICMP Protocol

### How `ping` works:
```
Your Machine                    Target Host
    │                               │
    ├──► ICMP Echo Request (Type 8) ──►│
    │                               │
    │◄── ICMP Echo Reply (Type 0) ◄──┤
    │                               │
    └── RTT = time between send & receive
```

### ICMP (Internet Control Message Protocol):
- Layer 3 (Network layer) protocol
- **NOT TCP or UDP** — no ports involved
- Used for diagnostics and error reporting
- Can be blocked by firewalls (host may be alive but not responding)

### Key metrics:
| Metric | Meaning | Good Value |
|--------|---------|-----------|
| **RTT** (Round-Trip Time) | Time for packet to go and return | < 50ms (LAN), < 200ms (internet) |
| **TTL** (Time to Live) | Max hops before packet dies | 64 (Linux), 128 (Windows), 255 (Cisco) |
| **Packet Loss** | % of packets that didn't return | 0% ideal, > 5% = problem |
| **Jitter** | Variation in latency | Low = stable connection |

### TTL for OS fingerprinting 🔒:
| TTL Value | Likely OS |
|-----------|----------|
| 64 | Linux/macOS |
| 128 | Windows |
| 255 | Cisco/Network device |

---

## 🧰 Syntax & Options

```bash
ping [OPTIONS] DESTINATION
```

| Flag | Description |
|------|-------------|
| `-c COUNT` | Send COUNT packets then stop |
| `-i SECS` | Interval between packets (default: 1s) |
| `-w TIMEOUT` | Deadline — stop after TIMEOUT seconds |
| `-W TIMEOUT` | Wait time for each reply |
| `-s SIZE` | Packet size in bytes (default: 56) |
| `-t TTL` | Set Time to Live |
| `-f` | **Flood ping** — send as fast as possible (root only) |
| `-q` | Quiet — only show summary |
| `-4` | Force IPv4 |
| `-6` | Force IPv6 |
| `-I INTERFACE` | Use specific network interface |
| `-D` | Print timestamps |
| `-n` | Numeric output (no DNS resolution) |
| `-a` | Audible ping (beep on reply) |

---

## 🟢 Basic Usage

```bash
# Ping a host (runs until Ctrl+C)
$ ping google.com
PING google.com (142.250.80.46) 56(84) bytes of data.
64 bytes from 142.250.80.46: icmp_seq=1 ttl=117 time=12.3 ms
64 bytes from 142.250.80.46: icmp_seq=2 ttl=117 time=11.8 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 11.8/12.05/12.3/0.25 ms

# Ping with count (stop after 5 packets)
$ ping -c 5 8.8.8.8

# Ping by IP (skip DNS)
$ ping -c 3 192.168.1.1

# Quick connectivity test
$ ping -c 1 -W 2 google.com && echo "Online" || echo "Offline"
```

---

## 🟡 Intermediate Usage

### Different packet sizes
```bash
# Small packet (fast)
$ ping -c 5 -s 16 192.168.1.1

# Large packet (test MTU)
$ ping -c 3 -s 1472 192.168.1.1       # Max without fragmentation (1500 MTU)

# Jumbo frame test
$ ping -c 3 -s 8972 192.168.1.1
```

### Fast/Flood ping (requires root)
```bash
# Flood ping — sends ASAP, shows only loss
$ sudo ping -f -c 1000 192.168.1.1
...
--- 192.168.1.1 ping statistics ---
1000 packets transmitted, 998 received, 0% packet loss

# Faster interval
$ ping -i 0.2 -c 20 192.168.1.1       # Every 200ms
```

### Timestamped output
```bash
$ ping -D google.com
[1716480000.123456] 64 bytes from 142.250.80.46: icmp_seq=1 ttl=117 time=12.3 ms
```

### Specific interface
```bash
$ ping -I eth0 google.com
$ ping -I wlan0 google.com
```

### Set TTL
```bash
# Short TTL — test how many hops to reach
$ ping -t 5 google.com
# If TTL expires → "Time to live exceeded"
```

---

## 🔴 Advanced Usage

### Network Troubleshooting Workflow
```bash
# Step 1: Ping localhost (loopback test)
$ ping -c 2 127.0.0.1            # Works? → Network stack OK

# Step 2: Ping gateway (local network)
$ ping -c 2 192.168.1.1          # Works? → LAN OK

# Step 3: Ping external IP (internet)
$ ping -c 2 8.8.8.8              # Works? → Internet OK

# Step 4: Ping domain name (DNS)
$ ping -c 2 google.com           # Works? → DNS OK

# If step 3 works but step 4 fails → DNS problem!
# If step 2 works but step 3 fails → Routing/ISP problem!
```

### Ping sweep (find alive hosts on network) 🔒
```bash
# Quick subnet scan
$ for i in $(seq 1 254); do
    ping -c 1 -W 1 192.168.1.$i > /dev/null 2>&1 && echo "192.168.1.$i is UP" &
done; wait

# Using fping (faster)
$ fping -a -g 192.168.1.0/24 2>/dev/null

# Using nmap (most reliable)
$ nmap -sn 192.168.1.0/24
```

### Monitoring script
```bash
#!/bin/bash
# ping_monitor.sh — Alert on connectivity loss
HOST="8.8.8.8"
LOG="/var/log/ping_monitor.log"

while true; do
    if ! ping -c 1 -W 3 "$HOST" > /dev/null 2>&1; then
        echo "$(date): ALERT — $HOST unreachable!" >> "$LOG"
    fi
    sleep 30
done
```

### Security considerations 🔒
```bash
# ICMP can be used for:
# - Host discovery (recon)
# - OS fingerprinting (TTL values)
# - ICMP tunneling (data exfiltration)
# - Ping of death (historical DoS)
# - Smurf attack (amplification)

# Check if ICMP is blocked:
$ ping -c 2 target.com
# No reply ≠ host is down (could be firewall!)

# ICMP tunneling tools: icmpsh, ptunnel, hans
```

---

## 🔗 Piping & Combining

```bash
# Ping and log only failures
$ ping -c 100 8.8.8.8 | grep -v "bytes from" >> failures.log

# Extract just RTT values
$ ping -c 10 google.com | grep "time=" | awk -F'time=' '{print $2}'

# Average latency
$ ping -c 10 -q google.com | tail -1 | awk -F'/' '{print "Avg:", $5, "ms"}'

# Ping multiple hosts
$ for host in google.com 8.8.8.8 cloudflare.com; do
    echo -n "$host: "
    ping -c 3 -q "$host" 2>/dev/null | tail -1 | awk -F'/' '{print $5, "ms"}' || echo "UNREACHABLE"
done
```

---

## 💡 Real World Pro Tips

### Tip 1: Quick online check in scripts
```bash
ping -c 1 -W 2 8.8.8.8 > /dev/null 2>&1 && echo "Online" || echo "Offline"
```

### Tip 2: Use `mtr` for better diagnostics
```bash
$ mtr google.com       # Combines ping + traceroute — shows per-hop loss
```

### Tip 3: No response ≠ host is down
```bash
# Many servers block ICMP — use other methods:
$ curl -s -o /dev/null -w "%{http_code}" https://target.com
$ nmap -Pn target.com    # Skip ping, scan directly
```

### Tip 4: OS detection via TTL
```bash
$ ping -c 1 target.com | grep ttl
# ttl=64 → Linux    ttl=128 → Windows    ttl=255 → Network device
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Simplest network diagnostic | ICMP often blocked by firewalls |
| Shows latency and packet loss | No response ≠ host down |
| Pre-installed everywhere | No port-level testing |
| OS fingerprinting via TTL | Can be used for DoS (flood) |

---

## 📍 Where & When to Use

| Scenario | Command | Alternative |
|----------|---------|-------------|
| Quick alive check | `ping -c 1 host` | `curl`, `nmap -sn` |
| Latency test | `ping -c 10 host` | `mtr` |
| Subnet discovery | ping sweep loop | `nmap -sn`, `fping` |
| Continuous monitor | `ping host` | `smokeping` |
| DNS test | `ping domain.com` | `dig`, `nslookup` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Assuming no reply = down | Could be firewall blocking ICMP |
| Forgetting `-c` (runs forever) | Always use `-c N` in scripts |
| Ignoring packet loss % | Even 1-2% loss indicates problems |
| Using ping for port testing | Use `nc`, `nmap`, or `curl` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Ping google.com with 5 packets
2. Ping your default gateway
3. Check the TTL value and guess the OS

### 🟡 Intermediate
4. Run the 4-step troubleshooting workflow
5. Write a ping sweep script for your subnet
6. Test MTU with large packet sizes

### 🔴 Advanced
7. Write a monitoring script that alerts on loss
8. Compare `ping` vs `mtr` vs `traceroute` output
9. Detect OS of 5 different hosts using TTL

---

## 🧠 Cheat Sheet

```
ping host                → Continuous ping
ping -c 5 host           → 5 packets
ping -c 1 -W 2 host      → Quick test (2s timeout)
ping -i 0.2 host         → Fast interval
ping -s 1472 host        → MTU test
ping -t 5 host           → Short TTL
sudo ping -f host        → Flood ping

TTL: 64=Linux  128=Windows  255=Cisco

TROUBLESHOOT:
  1. ping 127.0.0.1      → Loopback
  2. ping gateway        → LAN
  3. ping 8.8.8.8        → Internet
  4. ping google.com     → DNS

SWEEP:
  for i in $(seq 1 254); do ping -c1 -W1 192.168.1.$i &; done; wait
```

---

> **Previous**: [`cron/crontab` ←](../04_process_management/32_cron_crontab.md) | **Next**: [`ifconfig/ip` →](./34_ifconfig_ip.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
