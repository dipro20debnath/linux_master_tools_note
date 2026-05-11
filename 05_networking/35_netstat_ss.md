# 🛠️ `netstat` & `ss` — Network Statistics & Connections | Linux Master Note

> **See every network connection on your system. Who's connected, what ports are open, what's listening — the ultimate network visibility tool.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--sockets--connections)
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

### `netstat` vs `ss`:
| Feature | `netstat` | `ss` |
|---------|----------|------|
| Status | ⚠️ Deprecated (net-tools) | ✅ Modern (iproute2) |
| Speed | Slow (reads /proc) | **10x faster** (kernel netlink) |
| Output | Verbose | Concise |
| Availability | May not be installed | Pre-installed |

> 🎯 Use `ss` on modern systems. Know `netstat` for older systems and CTFs.

---

## 📖 Theory — Sockets & Connections

### TCP Connection States:
| State | Description |
|-------|-------------|
| `LISTEN` | Waiting for incoming connections |
| `ESTABLISHED` | Active connection |
| `SYN_SENT` | Connection request sent |
| `SYN_RECV` | Connection request received |
| `FIN_WAIT1/2` | Closing connection |
| `TIME_WAIT` | Waiting after close (2×MSL) |
| `CLOSE_WAIT` | Remote closed, waiting for local close |
| `CLOSED` | Connection terminated |

### Socket types:
| Type | Protocol | Use |
|------|----------|-----|
| TCP | Stream | Reliable connections (HTTP, SSH) |
| UDP | Datagram | Fast, connectionless (DNS, VPN) |
| UNIX | Local | Inter-process communication |
| RAW | Raw | Low-level (ping, custom packets) |

---

## 🧰 Syntax & Options

### Common flags (same for both):
| Flag | `netstat` | `ss` | Description |
|------|----------|------|-------------|
| `-t` | `-t` | `-t` | TCP connections |
| `-u` | `-u` | `-u` | UDP connections |
| `-l` | `-l` | `-l` | Listening sockets only |
| `-n` | `-n` | `-n` | Numeric (no DNS resolution) |
| `-p` | `-p` | `-p` | Show process name/PID |
| `-a` | `-a` | `-a` | All sockets |
| `-r` | `-r` | — | Routing table |
| `-s` | `-s` | `-s` | Summary statistics |

### The magic combo: `-tulnp`
```bash
$ sudo ss -tulnp       # TCP+UDP, Listening, Numeric, Process
$ sudo netstat -tulnp   # Same — the MOST USED combo ever
```

---

## 🟢 Basic Usage

```bash
# Show all listening ports (THE command you'll use most)
$ sudo ss -tulnp
State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
LISTEN  0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=1234))
LISTEN  0       511     0.0.0.0:80          0.0.0.0:*          users:(("nginx",pid=5678))
LISTEN  0       128     0.0.0.0:443         0.0.0.0:*          users:(("nginx",pid=5678))
LISTEN  0       80      127.0.0.1:3306      0.0.0.0:*          users:(("mysqld",pid=9012))

# netstat equivalent
$ sudo netstat -tulnp

# Show all connections (established + listening)
$ ss -tan

# Show established connections only
$ ss -t state established

# Quick: what ports are open?
$ ss -tln | awk 'NR>1 {print $4}' | sort -u
```

---

## 🟡 Intermediate Usage

### Filter by state
```bash
# Only established connections
$ ss -t state established

# Only listening
$ ss -t state listening

# TIME_WAIT connections (possible DoS or connection leak)
$ ss -t state time-wait | wc -l

# All states
$ ss -t state all
```

### Filter by port
```bash
# Connections on port 80
$ ss -tn sport = :80
$ ss -tn dport = :443

# Connections on port range
$ ss -tn sport \> :1024

# netstat equivalent
$ netstat -tan | grep ":80 "
```

### Filter by address
```bash
# Connections to specific IP
$ ss -tn dst 192.168.1.100

# Connections from specific subnet
$ ss -tn src 10.0.0.0/8
```

### Show process info
```bash
# Which process is using port 8080?
$ sudo ss -tlnp | grep 8080
LISTEN  0  128  *:8080  *:*  users:(("java",pid=1234,fd=12))

# Which process has the most connections?
$ sudo ss -tnp | awk '{print $NF}' | sort | uniq -c | sort -rn | head
```

### Statistics
```bash
$ ss -s
Total: 245
TCP:   42 (estab 15, closed 12, orphaned 0, timewait 8)
UDP:   8
```

---

## 🔴 Advanced Usage

### Security Auditing 🔒
```bash
# Find all open ports (attack surface!)
$ sudo ss -tulnp | awk 'NR>1 {print $1, $5}' | sort

# Find services listening on ALL interfaces (0.0.0.0 — exposed!)
$ sudo ss -tlnp | grep "0.0.0.0"

# Find services that should be local-only
# MySQL on 0.0.0.0 = DANGER!
$ sudo ss -tlnp | grep -E "(3306|5432|6379|27017)" | grep "0.0.0.0"

# Who is connected to SSH?
$ ss -tn state established sport = :22

# Count connections per source IP (DDoS detection)
$ ss -tn state established | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -10

# Find unusual outbound connections
$ ss -tnp | grep -v "127.0.0.1" | grep -v "::1"

# Check for reverse shells (common ports)
$ sudo ss -tnp | grep -E ":(4444|5555|1337|9001|9999) "
```

### Connection tracking for incident response
```bash
# Snapshot all connections with timestamps
$ date >> /tmp/conn_snapshot.txt
$ ss -tunap >> /tmp/conn_snapshot.txt

# Monitor new connections in real-time
$ watch -n 1 'ss -tn state established | wc -l'

# Find processes with most connections
$ sudo ss -tnp | awk -F'"' '{print $2}' | sort | uniq -c | sort -rn
```

### CTF/Pentesting — Port Enumeration 🎯
```bash
# After compromising a machine — internal recon
$ ss -tulnp              # What's running?
$ ss -tn state established  # Who's connected?

# Find internal services not visible from outside
$ ss -tlnp | grep "127.0.0.1"
# → Internal-only services = potential pivot targets

# Check for port forwarding
$ ss -tlnp | grep -v "127.0.0.1\|0.0.0.0"
```

---

## 💡 Real World Pro Tips

### Tip 1: The one command to rule them all
```bash
$ sudo ss -tulnp
# Shows ALL listening ports with process names — your go-to!
```

### Tip 2: Check if port is in use before starting service
```bash
$ ss -tln | grep ":8080 " && echo "Port in use!" || echo "Port free"
```

### Tip 3: Find which process uses a port
```bash
$ sudo ss -tlnp | grep ":80 "
# OR
$ sudo lsof -i :80
# OR
$ sudo fuser 80/tcp
```

### Tip 4: Monitor connection count
```bash
# Watch TCP state distribution
$ watch -n 2 'ss -s'

# Count connections per state
$ ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn
```

---

## ✅ Pros & Cons

| ✅ `ss` Pros | ❌ `netstat` Cons |
|-------------|------------------|
| 10x faster than netstat | Deprecated, may be removed |
| Kernel netlink API | Reads /proc (slow) |
| Advanced filtering | Limited filtering |
| State-based queries | Verbose output |
| Pre-installed everywhere | Requires net-tools package |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| What's listening? | `sudo ss -tulnp` | Show open ports |
| Who's connected? | `ss -tn state established` | Active connections |
| DDoS detection | Count per-IP connections | Find attackers |
| Port conflict | `ss -tln \| grep :PORT` | Check before starting |
| Internal recon (CTF) | `ss -tulnp` on target | Find services |
| Debug connection issues | `ss -s` | State distribution |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `netstat` on modern systems | Use `ss` (faster) |
| Forgetting `sudo` for `-p` flag | Process info needs root |
| Not using `-n` (slow DNS lookups) | Always add `-n` for speed |
| Ignoring `0.0.0.0` listeners | Services exposed to network! |
| Missing TIME_WAIT buildup | Monitor with `ss -s` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. List all listening ports with `ss -tulnp`
2. Find which process is using port 22
3. Count total established connections

### 🟡 Intermediate
4. Filter connections by state (established, time-wait)
5. Find the top 5 source IPs by connection count
6. Check if MySQL/Redis are exposed on 0.0.0.0

### 🔴 Advanced
7. Write a script to alert when connections exceed threshold
8. Detect potential reverse shells by checking suspicious ports
9. Monitor connection state distribution over time

---

## 🧠 Cheat Sheet

```
ESSENTIAL:
  sudo ss -tulnp           → ALL listening ports + process
  ss -tn state established  → Active connections
  ss -s                     → Summary statistics

FILTER:
  ss -tn sport = :80        → By source port
  ss -tn dst 10.0.0.1       → By destination
  ss -t state time-wait     → By state

SECURITY:
  ss -tlnp | grep 0.0.0.0   → Exposed services
  ss -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn
                             → Connections per IP

PORT CHECK:
  sudo ss -tlnp | grep :80  → Who uses port 80?
  sudo lsof -i :80          → Alternative
  sudo fuser 80/tcp         → Another alternative

LEGACY:
  sudo netstat -tulnp        → Same as ss -tulnp
```

---

> **Previous**: [`ifconfig/ip` ←](./34_ifconfig_ip.md) | **Next**: [`curl` →](./36_curl.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
