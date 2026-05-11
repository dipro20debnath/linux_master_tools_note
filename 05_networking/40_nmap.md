# 🛠️ `nmap` — Network Exploration & Security Scanner | Linux Master Note

> **The KING of network scanning. `nmap` discovers hosts, ports, services, OS, and vulnerabilities. THE most important tool for any cybersecurity professional.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--scanning-techniques)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [NSE Scripts](#-nse--nmap-scripting-engine)
8. [Real World Pro Tips](#-real-world-pro-tips)
9. [Pros & Cons](#-pros--cons)
10. [Where & When to Use](#-where--when-to-use)
11. [Common Mistakes](#-common-mistakes)
12. [Practice Exercises](#-practice-exercises)
13. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### What is nmap?
**N**etwork **Map**per — a free, open-source tool for network discovery and security auditing. Created by **Gordon Lyon (Fyodor)** in 1997.

### What nmap can do:
- **Host discovery** — Find alive hosts
- **Port scanning** — Find open ports
- **Service detection** — Identify running services + versions
- **OS detection** — Determine operating system
- **Vulnerability scanning** — NSE scripts for vuln detection
- **Firewall evasion** — Bypass IDS/IPS

---

## 📖 Theory — Scanning Techniques

### TCP 3-Way Handshake:
```
Client → SYN        → Server     (I want to connect)
Client ← SYN/ACK    ← Server     (OK, go ahead)
Client → ACK        → Server     (Connection established)
```

### Scan Types:
| Type | Flag | Method | Stealth | Speed |
|------|------|--------|---------|-------|
| **SYN scan** | `-sS` | Half-open (SYN→SYN/ACK→RST) | ✅ Stealthy | Fast |
| **Connect scan** | `-sT` | Full TCP connect | ❌ Logged | Slow |
| **UDP scan** | `-sU` | UDP packets | ✅ Stealthy | Very slow |
| **FIN scan** | `-sF` | FIN flag only | ✅ Stealthy | Moderate |
| **XMAS scan** | `-sX` | FIN+PSH+URG flags | ✅ Stealthy | Moderate |
| **NULL scan** | `-sN` | No flags | ✅ Stealthy | Moderate |
| **Ping scan** | `-sn` | ICMP/ARP only | — | Fast |
| **ACK scan** | `-sA` | ACK flag (firewall mapping) | — | Fast |

### Port States:
| State | Meaning |
|-------|---------|
| `open` | Service accepting connections |
| `closed` | Reachable but no service listening |
| `filtered` | Firewall blocking (can't determine) |
| `unfiltered` | Reachable but can't determine open/closed |
| `open\|filtered` | Can't determine which |

---

## 🧰 Syntax & Options

```bash
nmap [SCAN TYPE] [OPTIONS] TARGET(s)
```

### Target specification:
| Format | Example |
|--------|---------|
| Single IP | `192.168.1.1` |
| Range | `192.168.1.1-254` |
| CIDR | `192.168.1.0/24` |
| Hostname | `target.com` |
| From file | `-iL targets.txt` |
| Exclude | `--exclude 192.168.1.1` |

### Key options:
| Flag | Description |
|------|-------------|
| `-sS` | SYN scan (default, stealthy) |
| `-sT` | Connect scan (no root needed) |
| `-sU` | UDP scan |
| `-sn` | Ping scan (host discovery only) |
| `-sV` | **Service version detection** |
| `-sC` | **Default scripts** |
| `-O` | **OS detection** |
| `-A` | Aggressive (OS + version + scripts + traceroute) |
| `-p PORTS` | Port specification |
| `-p-` | **All 65535 ports** |
| `--top-ports N` | Top N most common ports |
| `-T0-5` | Timing (0=paranoid, 5=insane) |
| `-oN FILE` | Normal output |
| `-oX FILE` | XML output |
| `-oG FILE` | Grepable output |
| `-oA BASE` | All formats |
| `-v` / `-vv` | Verbose |
| `-Pn` | Skip host discovery (treat as up) |
| `--script=NAME` | Run NSE script |
| `-D decoy1,decoy2` | Decoy scan |
| `-S IP` | Spoof source IP |
| `-f` | Fragment packets |
| `--reason` | Show reason for port state |
| `--open` | Show only open ports |

---

## 🟢 Basic Usage

```bash
# Quick scan (top 1000 ports)
$ nmap 192.168.1.1

# Scan specific ports
$ nmap -p 22,80,443 192.168.1.1

# Scan port range
$ nmap -p 1-1000 192.168.1.1

# Scan ALL ports
$ nmap -p- 192.168.1.1

# Ping sweep (find alive hosts)
$ nmap -sn 192.168.1.0/24

# Service version detection
$ nmap -sV 192.168.1.1

# OS detection
$ sudo nmap -O 192.168.1.1

# Aggressive scan (version + OS + scripts)
$ sudo nmap -A 192.168.1.1
```

---

## 🟡 Intermediate Usage

### Service and version detection
```bash
# Detect service versions
$ nmap -sV -sC 192.168.1.1
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.9p1 Ubuntu
80/tcp  open  http     Apache httpd 2.4.52
443/tcp open  ssl/http nginx 1.22.0
3306/tcp open mysql    MySQL 8.0.33
```

### Timing templates
```bash
$ nmap -T0 target    # Paranoid (IDS evasion, very slow)
$ nmap -T1 target    # Sneaky
$ nmap -T2 target    # Polite
$ nmap -T3 target    # Normal (default)
$ nmap -T4 target    # Aggressive (recommended for CTF)
$ nmap -T5 target    # Insane (may miss results)
```

### Output formats
```bash
# Save in all formats
$ nmap -sV -sC -oA scan_results target

# Creates:
# scan_results.nmap   (normal)
# scan_results.xml    (XML)
# scan_results.gnmap  (grepable)

# Grepable — perfect for parsing
$ grep "open" scan_results.gnmap
```

### Scan multiple targets
```bash
# From file
$ cat targets.txt
192.168.1.1
192.168.1.50
10.10.10.100

$ nmap -iL targets.txt

# Subnet
$ nmap 192.168.1.0/24

# Range
$ nmap 192.168.1.1-50
```

### UDP scanning
```bash
# UDP scan (slow but important!)
$ sudo nmap -sU --top-ports 20 target

# Common UDP services: DNS(53), SNMP(161), TFTP(69), NTP(123)
$ sudo nmap -sU -p 53,161,69,123 target
```

---

## 🔴 Advanced Usage

### The CTF/Pentesting Standard Scan 🎯
```bash
# Step 1: Quick port discovery
$ nmap -p- --min-rate=1000 -T4 target -oN ports.txt
# Find ALL open ports fast

# Step 2: Detailed scan on found ports
$ nmap -p 22,80,443,8080 -sV -sC -A target -oA detailed

# Step 3: Vulnerability scan
$ nmap --script=vuln target
```

### Firewall evasion techniques
```bash
# Fragment packets
$ sudo nmap -f target

# Decoy scan (hide among decoys)
$ sudo nmap -D 10.0.0.1,10.0.0.2,ME target

# Custom source port
$ sudo nmap --source-port 53 target

# Slow scan to avoid IDS
$ nmap -T1 --max-rate 10 target

# Spoof MAC address
$ sudo nmap --spoof-mac Dell target
```

### Specific vulnerability checks
```bash
# Check for EternalBlue (MS17-010)
$ nmap --script smb-vuln-ms17-010 -p 445 target

# Check for Heartbleed
$ nmap --script ssl-heartbleed -p 443 target

# Check for ShellShock
$ nmap --script http-shellshock --script-args uri=/cgi-bin/test.cgi target

# Check for default credentials
$ nmap --script http-default-accounts target

# Full vulnerability scan
$ nmap --script vuln -sV target
```

---

## 📜 NSE — Nmap Scripting Engine

### Script categories:
| Category | Purpose |
|----------|---------|
| `auth` | Authentication bypass/check |
| `broadcast` | Network broadcast discovery |
| `brute` | Brute force attacks |
| `default` | Run with `-sC` |
| `discovery` | Service discovery |
| `exploit` | Active exploitation |
| `external` | Third-party queries |
| `fuzzer` | Fuzzing |
| `intrusive` | Potentially harmful |
| `malware` | Malware detection |
| `safe` | Won't crash targets |
| `version` | Version detection |
| `vuln` | Vulnerability detection |

### Useful NSE scripts:
```bash
# HTTP enumeration
$ nmap --script http-enum target

# DNS brute force
$ nmap --script dns-brute target

# SMB enumeration
$ nmap --script smb-enum-shares,smb-enum-users -p 445 target

# FTP anonymous login check
$ nmap --script ftp-anon -p 21 target

# SSL certificate info
$ nmap --script ssl-cert -p 443 target

# Banner grabbing
$ nmap --script banner -p 1-1000 target

# All safe scripts
$ nmap --script safe target
```

---

## 💡 Real World Pro Tips

### Tip 1: The 2-phase scan approach
```bash
# Phase 1: Fast all-port scan
$ nmap -p- --min-rate=5000 target -oG allports.txt
$ grep -oP '\d+/open' allports.txt | awk -F/ '{print $1}' | tr '\n' ','

# Phase 2: Deep scan on found ports
$ nmap -p 22,80,443,8080 -sV -sC -A target -oA detailed
```

### Tip 2: Only show open ports
```bash
$ nmap --open target
```

### Tip 3: Quick subnet alive check
```bash
$ nmap -sn 192.168.1.0/24 --open | grep "Nmap scan report"
```

### Tip 4: Save results ALWAYS
```bash
$ nmap -sV -sC target -oA scan_$(date +%Y%m%d)
# You'll want to reference these later!
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Most powerful scanner | Can be slow (full scan) |
| NSE scripting engine | Requires root for SYN scan |
| OS/Service detection | Detected by IDS/IPS |
| Multiple output formats | UDP scanning is very slow |
| Free and open source | Complex for beginners |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Host discovery | `nmap -sn subnet` | Find alive hosts |
| Quick port check | `nmap target` | Top 1000 ports |
| Full port scan | `nmap -p- target` | All 65535 ports |
| Service enumeration | `nmap -sV -sC target` | Version + scripts |
| Vulnerability scan | `nmap --script vuln target` | Find vulns |
| CTF initial recon | `nmap -p- --min-rate=5000` | Fast discovery |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Not scanning all ports | Use `-p-` for full scan |
| Running without root | `sudo nmap` for SYN scan |
| Not saving output | Always use `-oA filename` |
| Scanning too aggressively | Use `-T3` or `-T4`, not `-T5` |
| Ignoring UDP | Add `-sU` for UDP services |
| Forgetting version detection | Add `-sV -sC` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Scan your default gateway for open ports
2. Do a ping sweep of your local network
3. Scan a target with service version detection

### 🟡 Intermediate
4. Run a full port scan and save in all formats
5. Use NSE scripts to enumerate HTTP and SMB
6. Do a UDP scan on the top 20 ports

### 🔴 Advanced
7. Perform the 2-phase CTF scan approach
8. Use firewall evasion techniques
9. Run vulnerability scripts against a lab target

---

## 🧠 Cheat Sheet

```
HOST DISCOVERY:
  nmap -sn 192.168.1.0/24          → Ping sweep

PORT SCANNING:
  nmap target                       → Top 1000 ports
  nmap -p- target                   → ALL 65535 ports
  nmap -p 22,80,443 target          → Specific ports
  sudo nmap -sU target              → UDP scan

ENUMERATION:
  nmap -sV target                   → Service versions
  nmap -sC target                   → Default scripts
  nmap -A target                    → Aggressive (all)
  nmap -sV -sC -p- target          → Full enumeration

CTF WORKFLOW:
  nmap -p- --min-rate=5000 target   → Phase 1: find ports
  nmap -sV -sC -p PORTS target     → Phase 2: deep scan

NSE SCRIPTS:
  nmap --script vuln target         → Vulnerability scan
  nmap --script http-enum target    → HTTP directories
  nmap --script smb-enum-* target   → SMB enumeration

OUTPUT:
  -oN file.txt   → Normal
  -oX file.xml   → XML
  -oA basename   → All formats

TIMING: -T0 (paranoid) → -T4 (aggressive) → -T5 (insane)
EVASION: -f (fragment) -D decoys --source-port 53
```

---

> **Previous**: [`scp/rsync` ←](./39_scp_rsync.md) | **Next**: [`df` →](../06_disk_storage/41_df.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
