# 🛠️ `uname` — System Information | Linux Master Note

> **Know your system at a glance. `uname` reveals the kernel version, architecture, hostname, and OS — essential for troubleshooting, scripting, and security auditing.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--kernel--system-identity)
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

### What is `uname`?
`uname` (**U**nix **Name**) prints system information about the machine and OS. It reads directly from the kernel — fast and always accurate.

### Why it matters:
- **Troubleshooting** — Which kernel version is running?
- **Scripting** — Adapt scripts to different architectures
- **Security** — Check for vulnerable kernel versions
- **Compatibility** — Verify OS before installing software
- **CTF/Pentesting** — System reconnaissance on target

---

## 📖 Theory — Kernel & System Identity

### What `uname` reveals:
| Component | Example | Description |
|-----------|---------|-------------|
| Kernel name | `Linux` | OS kernel type |
| Hostname | `server01` | Network name of machine |
| Kernel release | `5.15.0-91-generic` | Kernel version + patch level |
| Kernel version | `#101-Ubuntu SMP` | Build info + date |
| Machine (arch) | `x86_64` | Hardware architecture |
| Processor | `x86_64` | Processor type |
| Platform | `x86_64` | Hardware platform |
| OS | `GNU/Linux` | Operating system |

### Architecture types:
| Architecture | Description |
|-------------|-------------|
| `x86_64` / `amd64` | 64-bit Intel/AMD |
| `i686` / `i386` | 32-bit Intel |
| `aarch64` / `arm64` | 64-bit ARM (RPi 4, M1) |
| `armv7l` | 32-bit ARM (RPi 3) |

---

## 🧰 Syntax & Options

```bash
uname [OPTIONS]
```

| Flag | Description | Example Output |
|------|-------------|----------------|
| `-s` | Kernel **name** | `Linux` |
| `-n` | **Hostname** (network node) | `server01` |
| `-r` | Kernel **release** | `5.15.0-91-generic` |
| `-v` | Kernel **version** (build) | `#101-Ubuntu SMP Tue...` |
| `-m` | **Machine** hardware | `x86_64` |
| `-p` | **Processor** type | `x86_64` |
| `-i` | Hardware **platform** | `x86_64` |
| `-o` | **Operating system** | `GNU/Linux` |
| `-a` | **ALL** information | Full one-liner |

---

## 🟢 Basic Usage

```bash
# Show ALL system info (most common)
$ uname -a
Linux server01 5.15.0-91-generic #101-Ubuntu SMP Tue Nov 14 13:30:08 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux

# Kernel name
$ uname -s
Linux

# Kernel release (version)
$ uname -r
5.15.0-91-generic

# Machine architecture
$ uname -m
x86_64

# Hostname
$ uname -n
server01

# Operating system
$ uname -o
GNU/Linux
```

---

## 🟡 Intermediate Usage

### Distro-specific info (beyond uname)
```bash
# Full OS release info
$ cat /etc/os-release
NAME="Ubuntu"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
ID=ubuntu
ID_LIKE=debian
VERSION_ID="22.04"
PRETTY_NAME="Ubuntu 22.04.3 LTS"

# Quick distro name
$ lsb_release -a 2>/dev/null
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.3 LTS
Release:        22.04
Codename:       jammy

# Kernel version details
$ cat /proc/version
Linux version 5.15.0-91-generic (buildd@lcy02-amd64-045)
```

### Architecture detection in scripts
```bash
#!/bin/bash
ARCH=$(uname -m)
case $ARCH in
    x86_64)  echo "64-bit Intel/AMD" ;;
    aarch64) echo "64-bit ARM" ;;
    armv7l)  echo "32-bit ARM" ;;
    i686)    echo "32-bit Intel" ;;
    *)       echo "Unknown: $ARCH" ;;
esac
```

### OS detection in scripts
```bash
#!/bin/bash
OS=$(uname -s)
case $OS in
    Linux)   echo "Linux system" ;;
    Darwin)  echo "macOS system" ;;
    CYGWIN*) echo "Cygwin on Windows" ;;
    MINGW*)  echo "MinGW on Windows" ;;
    *)       echo "Unknown: $OS" ;;
esac
```

---

## 🔴 Advanced Usage

### Security — Kernel Version Auditing 🔒
```bash
# Check current kernel
$ uname -r
5.15.0-91-generic

# Compare against known vulnerable kernels
# DirtyPipe: 5.8 - 5.16.10
# DirtyCow: 2.6.22 - 4.8.2
# PwnKit:   All versions with pkexec

# Check all installed kernels
$ dpkg -l | grep linux-image      # Debian/Ubuntu
$ rpm -qa | grep kernel           # RHEL/Fedora

# Check if kernel is up to date
$ apt list --upgradable 2>/dev/null | grep linux
```

### CTF/Pentesting — System Enumeration 🎯
```bash
# First things to run on a compromised box:
$ uname -a                        # Full system info
$ cat /etc/os-release             # Distro details
$ cat /proc/version               # Kernel build info
$ arch                            # Architecture shortcut
$ getconf LONG_BIT                # 32 or 64 bit

# Kernel exploit search pattern:
# 1. Get kernel version
$ uname -r
# 2. Search for exploits
# → searchsploit linux kernel 5.15
# → Google: "Linux kernel 5.15 privilege escalation"
```

### System info collection script
```bash
#!/bin/bash
# sysinfo.sh — Complete system fingerprint
echo "=== SYSTEM INFORMATION ==="
echo "Hostname:     $(uname -n)"
echo "Kernel:       $(uname -r)"
echo "Architecture: $(uname -m)"
echo "OS:           $(cat /etc/os-release | grep PRETTY_NAME | cut -d= -f2 | tr -d '\"')"
echo "CPU:          $(lscpu | grep 'Model name' | cut -d: -f2 | xargs)"
echo "Cores:        $(nproc)"
echo "RAM:          $(free -h | awk '/Mem:/ {print $2}')"
echo "Disk:         $(df -h / | awk 'NR==2 {print $2, "total,", $5, "used"}')"
echo "Uptime:       $(uptime -p)"
echo "IP:           $(hostname -I | awk '{print $1}')"
echo "Users:        $(who | wc -l) logged in"
```

---

## 🔗 Piping & Combining

```bash
# Kernel version only (for comparison)
$ uname -r | cut -d- -f1
5.15.0

# Check if 64-bit
$ [ "$(uname -m)" = "x86_64" ] && echo "64-bit" || echo "32-bit"

# Download correct binary for architecture
$ ARCH=$(uname -m)
$ wget "https://example.com/tool-${ARCH}" -O /usr/local/bin/tool
```

---

## 💡 Real World Pro Tips

### Tip 1: `uname -r` is your kernel fingerprint
```bash
# For installing kernel headers/modules:
$ sudo apt install linux-headers-$(uname -r)
```

### Tip 2: Use `/etc/os-release` for distro info
```bash
# uname doesn't tell you Ubuntu vs Debian vs Fedora
# Use this instead:
$ source /etc/os-release && echo "$NAME $VERSION"
```

### Tip 3: `arch` is a shortcut for `uname -m`
```bash
$ arch
x86_64
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Instant, always available | Doesn't show distro name |
| Reads from kernel (accurate) | Limited to kernel info |
| Great for scripting | No hardware details |
| Cross-platform (all Unix) | Architecture naming varies |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Quick system check | `uname -a` | Full overview |
| Kernel version | `uname -r` | Troubleshooting/security |
| Architecture check | `uname -m` | Download correct binary |
| Script portability | `uname -s` | OS detection |
| Install kernel headers | `linux-headers-$(uname -r)` | Exact match |
| CTF recon | `uname -a` + `/etc/os-release` | Exploit search |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Thinking uname shows distro | Use `/etc/os-release` |
| Confusing `-r` and `-v` | `-r` = version number, `-v` = build info |
| Not checking architecture before install | Always `uname -m` first |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Run `uname -a` and identify each field
2. Find your kernel version with `uname -r`
3. Check your system architecture

### 🟡 Intermediate
4. Write a script that adapts behavior based on OS/arch
5. Find your full distro info with `/etc/os-release`
6. Install kernel headers matching your kernel

### 🔴 Advanced
7. Write a full system info collection script
8. Check your kernel for known vulnerabilities
9. Compare `uname` output across different systems

---

## 🧠 Cheat Sheet

```
uname -a       → ALL system info
uname -s       → Kernel name (Linux)
uname -r       → Kernel release (5.15.0-91-generic)
uname -v       → Kernel build version
uname -m       → Architecture (x86_64)
uname -n       → Hostname
uname -o       → OS (GNU/Linux)
arch           → Shortcut for uname -m

DISTRO INFO:
  cat /etc/os-release
  lsb_release -a
  cat /proc/version

SCRIPTING:
  ARCH=$(uname -m)
  KERNEL=$(uname -r)
  sudo apt install linux-headers-$(uname -r)
```

---

> **Previous**: [`pip` ←](../07_package_management/49_pip.md) | **Next**: [`hostname` →](./51_hostname.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
