# 🛠️ `hostname` & `hostnamectl` — Show/Set System Hostname | Linux Master Note

> **Your system's identity on the network. `hostname` and `hostnamectl` let you view and control the machine's name — critical for networking, DNS, logging, and security reconnaissance.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory — Hostnames & Identity](#-theory--hostnames--identity)
3. [Syntax & Options — `hostname`](#-syntax--options--hostname)
4. [Syntax & Options — `hostnamectl`](#-syntax--options--hostnamectl)
5. [Basic Usage](#-basic-usage)
6. [Intermediate Usage](#-intermediate-usage)
7. [Advanced Usage](#-advanced-usage)
8. [Security & Pentesting](#-security--pentesting)
9. [Real World Pro Tips](#-real-world-pro-tips)
10. [Pros & Cons](#-pros--cons)
11. [Where & When to Use](#-where--when-to-use)
12. [Common Mistakes](#-common-mistakes)
13. [Practice Exercises](#-practice-exercises)
14. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### What is a Hostname?
A **hostname** is a human-readable label assigned to a machine on a network. Instead of remembering `192.168.1.42`, you use `webserver01`. It's the system's **identity** — appearing in shell prompts, logs, network traffic, and DNS.

### `hostname` vs `hostnamectl`
| Tool | Era | Init System | Persistence | Features |
|------|-----|-------------|-------------|----------|
| `hostname` | Legacy (pre-systemd) | SysVinit, any | Temporary unless you edit files | Basic get/set |
| `hostnamectl` | Modern (systemd) | systemd only | Persistent by default | Static, transient, pretty + metadata |

### Why it matters:
- **Networking** — Identify machines on LAN/WAN
- **DNS** — Hostname resolves to IP and vice versa
- **Logging** — syslog/journald tags every log with hostname
- **Automation** — Ansible, Terraform, cloud-init use hostnames
- **Security** — Hostname enumeration is step one of recon
- **Compliance** — Audit trails require proper machine identification

---

## 📖 Theory — Hostnames & Identity

### The Three Types of Hostnames (systemd)

| Type | Description | Where Stored | Example |
|------|-------------|-------------|---------|
| **Static** | Persistent name, survives reboot | `/etc/hostname` | `web-prod-01` |
| **Transient** | Temporary, set by DHCP or kernel | In-memory (kernel) | `dhcp-client-192` |
| **Pretty** | Free-form UTF-8, for display only | `/etc/machine-info` | `Dipro's Dev Laptop 🖥️` |

```
┌─────────────────────────────────────────────────────┐
│               HOSTNAME HIERARCHY                     │
├─────────────────────────────────────────────────────┤
│                                                      │
│   Pretty Hostname ──→  "Dipro's Web Server 🌐"      │
│   (Display only, UTF-8, stored in machine-info)      │
│                                                      │
│   Static Hostname ──→  "web-prod-01"                 │
│   (Persistent, /etc/hostname, RFC 1123 compliant)    │
│                                                      │
│   Transient Hostname ──→  "web-prod-01"              │
│   (In-memory, mirrors static unless overridden)      │
│                                                      │
│   FQDN ──→  "web-prod-01.example.com"               │
│   (Hostname + domain, resolved via DNS/hosts)        │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Hostname Rules (RFC 1123)
| Rule | Detail |
|------|--------|
| Max length | 64 characters (253 for FQDN) |
| Allowed chars | `a-z`, `0-9`, hyphen (`-`) |
| Cannot start/end with | Hyphen (`-`) |
| Case | Case-insensitive (stored lowercase) |
| No spaces | Use hyphens or underscores |
| No special chars | Pretty hostname is the exception |

### Key Configuration Files

| File | Purpose | Example Content |
|------|---------|-----------------|
| `/etc/hostname` | Static hostname (single line) | `web-prod-01` |
| `/etc/hosts` | Local hostname-to-IP mapping | `127.0.1.1 web-prod-01` |
| `/etc/machine-info` | Pretty hostname + metadata | `PRETTY_HOSTNAME="Dev Server"` |
| `/etc/nsswitch.conf` | Name resolution order | `hosts: files dns myhostname` |

### How Hostname Resolution Works

```
┌──────────────────────────────────────────────┐
│          HOSTNAME RESOLUTION ORDER            │
├──────────────────────────────────────────────┤
│                                               │
│  1. /etc/hosts         (local file lookup)    │
│          ↓ not found                          │
│  2. DNS Server         (network query)        │
│          ↓ not found                          │
│  3. mDNS/LLMNR        (local network)        │
│          ↓ not found                          │
│  4. nss-myhostname     (systemd fallback)     │
│                                               │
│  Order configured in: /etc/nsswitch.conf      │
│  hosts: files dns myhostname                  │
└──────────────────────────────────────────────┘
```

---

## 🧰 Syntax & Options — `hostname`

```bash
hostname [OPTIONS] [NEW_HOSTNAME]
```

| Flag | Description | Example |
|------|-------------|---------|
| *(none)* | Show current hostname | `hostname` → `server01` |
| `-s` | **Short** hostname (strip domain) | `hostname -s` → `server01` |
| `-f` / `--fqdn` | **Fully Qualified Domain Name** | `hostname -f` → `server01.example.com` |
| `-d` / `--domain` | DNS **domain** name | `hostname -d` → `example.com` |
| `-i` | **IP address** of hostname | `hostname -i` → `127.0.1.1` |
| `-I` | **All IP addresses** of the host | `hostname -I` → `192.168.1.42 10.0.0.5` |
| `-A` | **All FQDNs** of the host | `hostname -A` → `server01.example.com` |
| `-b` | Set hostname, use **default** if empty | `hostname -b server01` |
| `-F FILE` | Read hostname from **file** | `hostname -F /etc/hostname` |

> ⚠️ Running `hostname NEW_NAME` sets it **temporarily** (transient). It resets on reboot unless `/etc/hostname` is also updated.

---

## 🧰 Syntax & Options — `hostnamectl`

```bash
hostnamectl [COMMAND] [OPTIONS]
```

### Commands

| Command | Description | Example |
|---------|-------------|---------|
| `status` | Show all hostname info + OS metadata | `hostnamectl status` |
| `hostname [NAME]` | Get/set **static** hostname | `hostnamectl hostname web01` |
| `set-hostname NAME` | Set **static** hostname (legacy syntax) | `hostnamectl set-hostname web01` |

### Options

| Flag | Description | Example |
|------|-------------|---------|
| `--static` | Target **static** hostname only | `hostnamectl set-hostname web01 --static` |
| `--transient` | Target **transient** hostname only | `hostnamectl set-hostname temp01 --transient` |
| `--pretty` | Target **pretty** hostname only | `hostnamectl set-hostname "Dev Server 🖥️" --pretty` |
| `-H USER@HOST` | Execute on **remote** host via SSH | `hostnamectl -H root@10.0.0.5 status` |
| `--json=pretty` | Output in **JSON** format | `hostnamectl status --json=pretty` |
| `--no-ask-password` | Don't prompt for auth | Non-interactive scripts |

### `hostnamectl status` Output Explained

```bash
$ hostnamectl status
   Static hostname: web-prod-01
   Pretty hostname: Dipro's Web Server 🌐
         Icon name: computer-server
           Chassis: server
        Machine ID: 4a5b6c7d8e9f...
           Boot ID: 1a2b3c4d5e6f...
  Operating System: Ubuntu 22.04.3 LTS
            Kernel: Linux 5.15.0-91-generic
      Architecture: x86-64
   Firmware Version: 2.17
    Firmware Vendor: Dell Inc.
```

---

## 🟢 Basic Usage

### View Current Hostname
```bash
# Simple hostname
$ hostname
web-prod-01

# Full system identity (systemd)
$ hostnamectl
   Static hostname: web-prod-01
         Icon name: computer-server
           Chassis: server
  Operating System: Ubuntu 22.04.3 LTS
            Kernel: Linux 5.15.0-91-generic
      Architecture: x86-64

# Short hostname (no domain)
$ hostname -s
web-prod-01

# FQDN (Fully Qualified Domain Name)
$ hostname -f
web-prod-01.example.com

# Domain name
$ hostname -d
example.com
```

### View IP Addresses
```bash
# All IPs assigned to this host
$ hostname -I
192.168.1.42 10.0.0.5 172.17.0.1

# IP associated with hostname (from /etc/hosts)
$ hostname -i
127.0.1.1
```

### Set Hostname (Temporary)
```bash
# Temporary — resets after reboot
$ sudo hostname new-server-name
```

### Set Hostname (Persistent — Recommended)
```bash
# Persistent — survives reboot (systemd)
$ sudo hostnamectl set-hostname new-server-name
```

---

## 🟡 Intermediate Usage

### Set All Three Hostname Types
```bash
# Set static hostname (persistent, RFC-compliant)
$ sudo hostnamectl set-hostname web-prod-01 --static

# Set transient hostname (temporary, runtime only)
$ sudo hostnamectl set-hostname temp-testing --transient

# Set pretty hostname (display name, UTF-8 allowed)
$ sudo hostnamectl set-hostname "Dipro's Production Web Server 🌐" --pretty

# Verify all three
$ hostnamectl
   Static hostname: web-prod-01
Transient hostname: temp-testing
   Pretty hostname: Dipro's Production Web Server 🌐
```

### Configure `/etc/hostname`
```bash
# View current static hostname file
$ cat /etc/hostname
web-prod-01

# Manually change (alternative to hostnamectl)
$ echo "new-hostname" | sudo tee /etc/hostname

# Apply without reboot
$ sudo hostname -F /etc/hostname
```

### Configure `/etc/hosts`
```bash
# Always update /etc/hosts when changing hostname!
$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       web-prod-01.example.com  web-prod-01

# Edit with sed
$ sudo sed -i 's/old-hostname/new-hostname/g' /etc/hosts
```

### Hostname in Shell Prompt
```bash
# PS1 uses these escape sequences:
#   \h = short hostname    \H = FQDN
$ echo $PS1
\u@\h:\w\$
# Displays as: dipro@web-prod-01:~$

# Change prompt to show FQDN
export PS1='\u@\H:\w\$ '
# Displays as: dipro@web-prod-01.example.com:~$
```

### Check FQDN Setup
```bash
# Verify FQDN is properly configured
$ hostname -f
web-prod-01.example.com

# If FQDN fails, check /etc/hosts:
# The FQDN MUST be the first entry after the IP
# ✅ Correct:
127.0.1.1   web-prod-01.example.com  web-prod-01

# ❌ Wrong (short name first):
127.0.1.1   web-prod-01  web-prod-01.example.com
```

---

## 🔴 Advanced Usage

### Complete Hostname Change Procedure (No Reboot)
```bash
#!/bin/bash
# change-hostname.sh — Change hostname properly without reboot
# Usage: sudo ./change-hostname.sh <new-hostname> [domain]

NEW_HOST="$1"
DOMAIN="${2:-localdomain}"
OLD_HOST=$(hostname)
FQDN="${NEW_HOST}.${DOMAIN}"

if [ -z "$NEW_HOST" ]; then
    echo "Usage: $0 <new-hostname> [domain]"
    exit 1
fi

echo "[*] Changing hostname: $OLD_HOST → $NEW_HOST"

# Step 1: Set static hostname (systemd)
hostnamectl set-hostname "$NEW_HOST" --static
echo "[✓] Static hostname set"

# Step 2: Set transient hostname
hostnamectl set-hostname "$NEW_HOST" --transient
echo "[✓] Transient hostname set"

# Step 3: Set pretty hostname
hostnamectl set-hostname "$NEW_HOST" --pretty
echo "[✓] Pretty hostname set"

# Step 4: Update /etc/hosts
sed -i "s/$OLD_HOST/$NEW_HOST/g" /etc/hosts
# Ensure FQDN entry exists
grep -q "$FQDN" /etc/hosts || \
    sed -i "s/127.0.1.1.*/127.0.1.1\t$FQDN\t$NEW_HOST/" /etc/hosts
echo "[✓] /etc/hosts updated"

# Step 5: Update /etc/hostname
echo "$NEW_HOST" > /etc/hostname
echo "[✓] /etc/hostname updated"

# Step 6: Restart systemd-hostnamed
systemctl restart systemd-hostnamed 2>/dev/null
echo "[✓] systemd-hostnamed restarted"

# Verify
echo ""
echo "=== VERIFICATION ==="
echo "hostname:     $(hostname)"
echo "hostname -f:  $(hostname -f)"
echo "hostnamectl:"
hostnamectl --static
```

### Remote Hostname Management
```bash
# Check hostname on remote server
$ hostnamectl -H root@10.0.0.5 status

# Set hostname on remote server
$ hostnamectl -H root@10.0.0.5 set-hostname db-replica-02

# Batch rename servers via SSH
for i in {1..5}; do
    ssh root@10.0.0.$((10+i)) "hostnamectl set-hostname node-0${i}"
    echo "Set node-0${i} on 10.0.0.$((10+i))"
done
```

### Ansible Hostname Configuration
```yaml
# playbook: set-hostnames.yml
---
- name: Configure server hostnames
  hosts: all
  become: yes
  tasks:
    - name: Set hostname
      ansible.builtin.hostname:
        name: "{{ inventory_hostname }}"
        use: systemd

    - name: Update /etc/hosts
      ansible.builtin.lineinfile:
        path: /etc/hosts
        regexp: '^127\.0\.1\.1'
        line: "127.0.1.1\t{{ inventory_hostname }}.{{ domain }}\t{{ inventory_hostname }}"
```

### Hostname-Based Conditional Scripts
```bash
#!/bin/bash
# Run different configs based on hostname
HOSTNAME=$(hostname -s)

case "$HOSTNAME" in
    web-*)
        echo "Starting Nginx..."
        systemctl start nginx
        ;;
    db-*)
        echo "Starting PostgreSQL..."
        systemctl start postgresql
        ;;
    cache-*)
        echo "Starting Redis..."
        systemctl start redis
        ;;
    *)
        echo "Unknown server role for: $HOSTNAME"
        ;;
esac
```

### Docker & Container Hostnames
```bash
# Docker sets hostname = container ID by default
$ docker run --hostname myapp-container alpine hostname
myapp-container

# Docker Compose
# docker-compose.yml:
#   services:
#     web:
#       hostname: web-frontend
#       domainname: app.local

# Check hostname inside container
$ docker exec -it myapp hostname -f
myapp-container.app.local

# Kubernetes pod hostname
$ kubectl exec -it mypod -- hostname
mypod
```

---

## 🔒 Security & Pentesting

### 🎯 Hostname Enumeration in CTF/Pentesting
```bash
# === INITIAL RECONNAISSANCE ===

# Step 1: Get hostname on compromised target
$ hostname
corp-dc-01

# Step 2: Full system identity
$ hostnamectl
   Static hostname: corp-dc-01
  Operating System: Ubuntu 22.04 LTS
            Kernel: Linux 5.15.0-91-generic
      Architecture: x86-64

# Step 3: Get FQDN — reveals domain!
$ hostname -f
corp-dc-01.internal.megacorp.com
# ↑ Now you know the internal domain: internal.megacorp.com

# Step 4: Get all IPs — find other interfaces
$ hostname -I
192.168.1.10 10.10.0.5 172.16.0.1
# ↑ Multiple interfaces = potential pivot points

# Step 5: Read hosts file — find other hosts
$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       corp-dc-01.internal.megacorp.com corp-dc-01
10.10.0.1       corp-db-01.internal.megacorp.com
10.10.0.2       corp-web-01.internal.megacorp.com
10.10.0.3       corp-git-01.internal.megacorp.com
# ↑ Goldmine! Internal network map discovered
```

### 🔍 Network-Based Hostname Discovery
```bash
# Discover hostnames on the network
# Reverse DNS lookup
$ nmap -sn 192.168.1.0/24 | grep -E "scan|Host"
Nmap scan report for corp-dc-01.local (192.168.1.10)
Host is up (0.001s latency).
Nmap scan report for corp-web-01.local (192.168.1.20)
Host is up (0.002s latency).

# NetBIOS hostname enumeration (Windows)
$ nbtscan 192.168.1.0/24
192.168.1.10   CORP-DC-01      MEGACORP\DC

# DNS zone transfer attempt (if misconfigured)
$ dig axfr internal.megacorp.com @10.10.0.1

# mDNS discovery
$ avahi-browse -art
```

### 🛡️ Hostname Hardening
```bash
# 1. Don't reveal role in hostname (attackers love "dc", "db", "admin")
# ❌ Bad:  domain-controller-01, sql-db-master
# ✅ Good: srv-alpha-01, node-bravo-02

# 2. Disable hostname in HTTP headers
# Apache: ServerTokens Prod
# Nginx:  server_tokens off;

# 3. Remove hostname from SMTP banner
# Postfix: smtpd_banner = $myhostname ESMTP
# Change to generic: smtpd_banner = mail ESMTP

# 4. Don't expose hostname in DNS unnecessarily
# Restrict zone transfers:
# /etc/bind/named.conf:
#   allow-transfer { none; };

# 5. Monitor hostname changes (detect tampering)
$ inotifywait -m /etc/hostname
# Or audit with auditd:
$ sudo auditctl -w /etc/hostname -p wa -k hostname_change
```

### 🏁 CTF Quick Enumeration Script
```bash
#!/bin/bash
# ctf-hostname-enum.sh — Hostname-based intel gathering
echo "=== HOSTNAME INTEL ==="
echo "Hostname:    $(hostname 2>/dev/null)"
echo "FQDN:        $(hostname -f 2>/dev/null)"
echo "Domain:      $(hostname -d 2>/dev/null)"
echo "All IPs:     $(hostname -I 2>/dev/null)"
echo ""
echo "=== /etc/hostname ==="
cat /etc/hostname 2>/dev/null
echo ""
echo "=== /etc/hosts (non-localhost) ==="
grep -v '^\s*#\|localhost\|^$' /etc/hosts 2>/dev/null
echo ""
echo "=== /etc/resolv.conf ==="
cat /etc/resolv.conf 2>/dev/null
echo ""
echo "=== DNS Domain Search ==="
grep "search\|domain" /etc/resolv.conf 2>/dev/null
echo ""
echo "=== Interesting hostnames nearby ==="
arp -a 2>/dev/null | head -20
```

---

## 💡 Real World Pro Tips

### Tip 1: Change Hostname Without Reboot (One-Liner)
```bash
# The cleanest single command — sets all three + updates files
$ sudo hostnamectl set-hostname new-name && \
  sudo sed -i "s/$(hostname)/new-name/g" /etc/hosts
# Done! No reboot needed. New shell sessions show the new name.
```

### Tip 2: Cloud Instance Naming Best Practices
```bash
# AWS — Preserve hostname across reboots
# /etc/cloud/cloud.cfg:
#   preserve_hostname: true

# Set meaningful cloud hostnames
$ sudo hostnamectl set-hostname "web-prod-us-east-1a-001"
# Pattern: role-env-region-az-number

# AWS user-data script (runs at launch):
#!/bin/bash
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/region)
hostnamectl set-hostname "web-${INSTANCE_ID}-${REGION}"

# GCP — Hostname from metadata
$ curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/hostname
```

### Tip 3: Hostname in Logging & Monitoring
```bash
# Syslog uses hostname for every log entry
# Bad hostname = unreadable logs
$ journalctl | head -3
May 26 01:00:01 web-prod-01 systemd[1]: Started timer.
May 26 01:00:02 web-prod-01 CRON[1234]: (root) CMD (...)

# Centralized logging (ELK/Splunk) groups by hostname
# Duplicate hostnames = merged logs = nightmare
# Always use UNIQUE hostnames!
```

### Tip 4: Quick Hostname Check in Scripts
```bash
# Multiple ways to get hostname — pick the right one
hostname            # Standard command
hostname -s         # Short (no domain)
hostname -f         # FQDN
cat /etc/hostname   # From file
echo $HOSTNAME      # Bash variable (cached at login)
uname -n            # From kernel

# ⚠️ $HOSTNAME may be stale if changed without new login
# Prefer $(hostname) in scripts for accuracy
```

### Tip 5: WSL/VM Hostname Sync
```bash
# WSL inherits Windows hostname by default
# To set a custom WSL hostname, edit /etc/wsl.conf:
[network]
hostname = my-wsl-dev

# Then restart WSL:
# (PowerShell) wsl --shutdown

# VirtualBox — set hostname via cloud-init or provisioning
```

---

## ✅ Pros & Cons

### `hostname` (Legacy)

| ✅ Pros | ❌ Cons |
|---------|---------|
| Available on all Unix/Linux | Changes are **temporary** by default |
| Simple, no dependencies | Must manually edit files for persistence |
| Works without systemd | No concept of pretty hostname |
| Fast, lightweight | No OS/hardware metadata |

### `hostnamectl` (Modern)

| ✅ Pros | ❌ Cons |
|---------|---------|
| **Persistent** by default | Requires **systemd** |
| Manages 3 hostname types | Not available on Alpine/BusyBox |
| Shows OS, kernel, architecture | Slightly more complex syntax |
| Remote host support (`-H`) | Newer — not on legacy systems |
| JSON output for automation | Overkill for quick checks |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Quick hostname check | `hostname` | Fastest method |
| Full system identity | `hostnamectl` | Shows hostname + OS + kernel |
| Get all IPs | `hostname -I` | Find network interfaces |
| Get FQDN | `hostname -f` | DNS/mail configuration |
| Set permanent hostname | `hostnamectl set-hostname` | Survives reboot |
| Set display name | `hostnamectl --pretty` | Nice name with emoji/spaces |
| Server provisioning | `hostnamectl` + `/etc/hosts` | Proper setup |
| CTF/Pentesting recon | `hostname -f` + `/etc/hosts` | Domain & network discovery |
| Container naming | `docker --hostname` | Identify containers |
| Cloud instances | cloud-init + `hostnamectl` | Automated naming |

---

## ⚠️ Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| `sudo hostname newname` only | Resets after reboot | Use `hostnamectl set-hostname` |
| Not updating `/etc/hosts` | `hostname -f` fails, apps break | Always sync `/etc/hosts` |
| FQDN not first in `/etc/hosts` | `hostname -f` returns short name | Put `FQDN` before short name |
| Using `$HOSTNAME` in scripts | Variable cached at login, may be stale | Use `$(hostname)` instead |
| Spaces/special chars in hostname | Network failures, DNS issues | Only `a-z`, `0-9`, `-` for static |
| Duplicate hostnames on network | Log confusion, DNS conflicts | Always use unique names |
| Forgetting cloud `preserve_hostname` | Cloud-init resets hostname on reboot | Set `preserve_hostname: true` |
| Not restarting services after change | Some services cache old hostname | Restart syslog, nginx, etc. |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Run `hostname` and `hostnamectl` — compare the output
2. Find your system's FQDN with `hostname -f`
3. List all IP addresses with `hostname -I`
4. Read the contents of `/etc/hostname` and `/etc/hosts`
5. Check what your shell prompt shows (`echo $PS1`) — find the hostname escape

### 🟡 Intermediate
6. Change your hostname temporarily with `hostname`, then verify it resets after reboot
7. Use `hostnamectl set-hostname` to set a persistent hostname — verify with `cat /etc/hostname`
8. Set a pretty hostname with emoji: `hostnamectl set-hostname "My Dev Box 🖥️" --pretty`
9. Update `/etc/hosts` to match your new hostname and verify `hostname -f` works
10. Write a script that prints hostname, FQDN, domain, and all IPs in a formatted output

### 🔴 Advanced
11. Write a complete hostname change script that updates all files without reboot
12. Set up hostname-based conditional service startup (web servers vs. db servers)
13. Use `hostnamectl -H` to check/set hostname on a remote machine
14. Write a CTF enumeration script that extracts hostname intel from a target
15. Configure cloud-init to auto-name instances with role-region-id pattern

---

## 🧠 Cheat Sheet

```
============================================
  hostname & hostnamectl — Quick Reference
============================================

VIEW HOSTNAME:
  hostname              → Current hostname
  hostname -s           → Short hostname (no domain)
  hostname -f           → FQDN (host.domain.com)
  hostname -d           → Domain name only
  hostname -I           → All IP addresses
  hostname -i           → IP from /etc/hosts
  hostnamectl           → Full system identity
  cat /etc/hostname     → Stored static hostname

SET HOSTNAME:
  sudo hostname NAME               → Temporary (lost on reboot)
  sudo hostnamectl set-hostname NAME → Persistent (recommended)
  sudo hostnamectl set-hostname NAME --static     → Static only
  sudo hostnamectl set-hostname NAME --transient   → Transient only
  sudo hostnamectl set-hostname "NAME" --pretty    → Pretty (UTF-8)

CONFIGURATION FILES:
  /etc/hostname         → Static hostname (1 line)
  /etc/hosts            → Local DNS mapping
  /etc/machine-info     → Pretty hostname storage
  /etc/nsswitch.conf    → Resolution order
  /etc/cloud/cloud.cfg  → Cloud hostname settings

AFTER CHANGING HOSTNAME:
  1. hostnamectl set-hostname NEW_NAME
  2. Update /etc/hosts  (FQDN first, then short)
  3. Verify: hostname && hostname -f

REMOTE:
  hostnamectl -H user@host status
  hostnamectl -H user@host set-hostname NAME

SECURITY/RECON:
  hostname -f           → Reveals internal domain
  hostname -I           → Shows all interfaces
  cat /etc/hosts        → Internal host mapping
  cat /etc/resolv.conf  → DNS servers + domain

HOSTNAME TYPES:
  Static    → Permanent, in /etc/hostname
  Transient → Runtime, set by DHCP/kernel
  Pretty    → Display name, UTF-8 allowed
```

---

> **Previous**: [`uname` ←](./50_uname.md) | **Next**: [`dmesg` →](./52_dmesg.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
