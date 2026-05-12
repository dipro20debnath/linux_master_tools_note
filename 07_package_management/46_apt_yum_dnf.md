# 🛠️ `apt` / `yum` / `dnf` — Package Managers | Linux Master Note

> **Install, update, and remove software with a single command. Package managers are the app stores of Linux — the backbone of system administration.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--package-management)
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

### Which distro uses which?
| Distro Family | High-Level | Low-Level | Package Format |
|---------------|-----------|-----------|---------------|
| **Debian/Ubuntu** | `apt` / `apt-get` | `dpkg` | `.deb` |
| **RHEL/CentOS/Fedora** | `dnf` / `yum` | `rpm` | `.rpm` |
| **Arch Linux** | `pacman` | — | `.pkg.tar.zst` |
| **openSUSE** | `zypper` | `rpm` | `.rpm` |
| **Alpine** | `apk` | — | `.apk` |

### `apt` vs `apt-get`:
| Feature | `apt` | `apt-get` |
|---------|-------|-----------|
| Progress bar | ✅ Yes | ❌ No |
| Color output | ✅ Yes | ❌ No |
| Combines commands | ✅ (`apt list`) | ❌ Separate tools |
| Scripting | ⚠️ Output may change | ✅ Stable output |
| Recommendation | **Interactive use** | **Scripts** |

### `yum` vs `dnf`:
| Feature | `yum` | `dnf` |
|---------|-------|-------|
| Status | ⚠️ Legacy | ✅ Modern replacement |
| Speed | Slower | Faster (C library) |
| Dependency solver | Basic | Advanced (libsolv) |
| RHEL/CentOS 7 | ✅ Default | Available |
| RHEL 8+/Fedora | Alias to dnf | ✅ Default |

---

## 📖 Theory — Package Management

### How it works:
```
Repository (server)          Your System
┌─────────────────┐         ┌────────────────┐
│ Package Index    │ ──apt── │ Local Cache     │
│ .deb/.rpm files  │ update  │ /var/cache/apt/ │
│ Dependencies     │         │                │
└─────────────────┘         └────────────────┘
                                    │
                              apt install
                                    │
                            ┌───────▼────────┐
                            │ Installed Pkgs  │
                            │ /usr/bin/       │
                            │ /etc/           │
                            │ /lib/           │
                            └────────────────┘
```

### Repository sources:
```bash
# Debian/Ubuntu: /etc/apt/sources.list + /etc/apt/sources.list.d/
deb http://archive.ubuntu.com/ubuntu jammy main restricted
deb http://archive.ubuntu.com/ubuntu jammy universe multiverse

# RHEL/CentOS: /etc/yum.repos.d/*.repo
[baseos]
name=CentOS Stream BaseOS
baseurl=https://mirror.centos.org/centos-stream/
gpgcheck=1
```

### Package naming:
```
nginx_1.22.0-1ubuntu1_amd64.deb
│     │         │       │
│     │         │       └── Architecture
│     │         └── Distribution revision
│     └── Version number
└── Package name
```

---

## 🧰 Syntax & Options

### APT (Debian/Ubuntu):
| Command | Description |
|---------|-------------|
| `apt update` | Refresh package index |
| `apt upgrade` | Upgrade installed packages |
| `apt full-upgrade` | Upgrade + remove obsolete |
| `apt install PKG` | Install package |
| `apt remove PKG` | Remove package (keep config) |
| `apt purge PKG` | Remove package + config files |
| `apt autoremove` | Remove unused dependencies |
| `apt search TERM` | Search for packages |
| `apt show PKG` | Package details |
| `apt list --installed` | List installed packages |
| `apt list --upgradable` | Show available upgrades |
| `apt edit-sources` | Edit sources.list |

### DNF/YUM (RHEL/Fedora):
| Command | Description |
|---------|-------------|
| `dnf check-update` | Check for updates |
| `dnf upgrade` / `dnf update` | Upgrade all packages |
| `dnf install PKG` | Install package |
| `dnf remove PKG` | Remove package |
| `dnf search TERM` | Search packages |
| `dnf info PKG` | Package details |
| `dnf list installed` | List installed |
| `dnf list available` | List available |
| `dnf provides FILE` | Find which package owns a file |
| `dnf group list` | List package groups |
| `dnf group install GROUP` | Install package group |
| `dnf history` | Transaction history |
| `dnf history undo N` | Undo transaction |
| `dnf clean all` | Clear cache |

---

## 🟢 Basic Usage

### Update & Upgrade
```bash
# Debian/Ubuntu
$ sudo apt update              # Refresh package list
$ sudo apt upgrade             # Upgrade packages
$ sudo apt update && sudo apt upgrade -y   # Combined

# RHEL/Fedora
$ sudo dnf check-update        # Check what can be upgraded
$ sudo dnf upgrade -y          # Upgrade all
```

### Install packages
```bash
# Debian/Ubuntu
$ sudo apt install nginx
$ sudo apt install nginx php mysql-server   # Multiple
$ sudo apt install -y nginx                 # Auto-yes

# RHEL/Fedora
$ sudo dnf install nginx
$ sudo dnf install -y nginx php mysql-server
```

### Remove packages
```bash
# Debian/Ubuntu
$ sudo apt remove nginx          # Keep config files
$ sudo apt purge nginx           # Remove everything
$ sudo apt autoremove            # Clean unused deps

# RHEL/Fedora
$ sudo dnf remove nginx
$ sudo dnf autoremove
```

### Search packages
```bash
# Debian/Ubuntu
$ apt search "web server"
$ apt show nginx

# RHEL/Fedora
$ dnf search "web server"
$ dnf info nginx
```

---

## 🟡 Intermediate Usage

### Install specific version
```bash
# Debian/Ubuntu
$ apt list -a nginx                    # Show available versions
$ sudo apt install nginx=1.22.0-1ubuntu1   # Specific version

# RHEL/Fedora
$ dnf list nginx --showduplicates
$ sudo dnf install nginx-1.22.0
```

### Hold/Pin package (prevent upgrades)
```bash
# Debian/Ubuntu
$ sudo apt-mark hold nginx             # Prevent upgrade
$ sudo apt-mark unhold nginx           # Allow upgrade
$ apt-mark showhold                    # List held packages

# RHEL/Fedora
$ sudo dnf versionlock add nginx
$ sudo dnf versionlock delete nginx
$ dnf versionlock list
```

### Find which package provides a file
```bash
# Debian/Ubuntu
$ apt-file search /usr/bin/curl        # Need: apt install apt-file
$ dpkg -S /usr/bin/curl                # For installed packages
curl: /usr/bin/curl

# RHEL/Fedora
$ dnf provides /usr/bin/curl
$ dnf provides "*/curl"
```

### List installed packages
```bash
# Debian/Ubuntu
$ apt list --installed
$ apt list --installed | wc -l         # Count
$ apt list --installed | grep nginx

# RHEL/Fedora
$ dnf list installed
$ rpm -qa | grep nginx
```

### Download without installing
```bash
# Debian/Ubuntu
$ apt download nginx                    # Downloads .deb
$ sudo apt install --download-only nginx  # Download to cache

# RHEL/Fedora
$ dnf download nginx                    # Downloads .rpm
$ dnf download --resolve nginx          # With dependencies
```

### Add PPA/Repository
```bash
# Debian/Ubuntu — PPA
$ sudo add-apt-repository ppa:ondrej/php
$ sudo apt update

# Debian/Ubuntu — Manual repo
$ echo "deb https://repo.example.com/apt stable main" | sudo tee /etc/apt/sources.list.d/example.list
$ wget -qO- https://repo.example.com/key.gpg | sudo apt-key add -
$ sudo apt update

# RHEL/Fedora — Add repo
$ sudo dnf config-manager --add-repo https://repo.example.com/example.repo
# Or install repo RPM
$ sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

---

## 🔴 Advanced Usage

### Security — Package Verification 🔒
```bash
# Verify installed packages (integrity check)
# Debian/Ubuntu
$ sudo debsums -c                      # Check for modified files
$ sudo debsums -s                      # Summary

# RHEL/Fedora
$ rpm -Va                              # Verify ALL packages
# S = size, M = mode, 5 = MD5, T = mtime changed

# Check package signatures
$ apt-key list                         # List trusted keys
$ rpm -K package.rpm                   # Verify RPM signature
```

### Transaction history (undo changes!)
```bash
# RHEL/Fedora (powerful feature!)
$ dnf history
  ID | Command              | Date and time    | Action(s)
   5 | install nginx        | 2026-05-12 10:00 | Install
   4 | upgrade              | 2026-05-11 02:00 | Upgrade
   
$ sudo dnf history undo 5              # Undo install nginx!
$ sudo dnf history redo 5              # Redo it

# Debian/Ubuntu — check apt logs
$ cat /var/log/apt/history.log
$ cat /var/log/dpkg.log
```

### Unattended upgrades (auto security patches)
```bash
# Debian/Ubuntu
$ sudo apt install unattended-upgrades
$ sudo dpkg-reconfigure -plow unattended-upgrades

# Config: /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
```

### CTF/Pentesting — Package Enumeration 🎯
```bash
# What tools are installed? (recon on compromised machine)
$ dpkg -l | grep -iE "nmap|wireshark|john|hashcat|hydra|netcat"
$ rpm -qa | grep -iE "nmap|wireshark|netcat"

# What packages were recently installed?
$ grep " install " /var/log/dpkg.log | tail -20
$ dnf history | head -20

# Install pentesting tools
$ sudo apt install nmap netcat-openbsd wireshark tcpdump
$ sudo dnf install nmap ncat wireshark tcpdump
```

### Cleanup & disk space recovery
```bash
# Debian/Ubuntu
$ sudo apt clean                       # Clear ALL cached .deb files
$ sudo apt autoclean                   # Clear old cached .deb files
$ sudo apt autoremove --purge          # Remove unused deps + config
$ du -sh /var/cache/apt/archives/      # Check cache size

# RHEL/Fedora
$ sudo dnf clean all
$ sudo dnf autoremove
$ du -sh /var/cache/dnf/
```

---

## 💡 Real World Pro Tips

### Tip 1: Always update before install
```bash
$ sudo apt update && sudo apt install nginx -y
# Ensures you get the latest version from repos
```

### Tip 2: Simulate before acting
```bash
# See what would happen without doing it
$ apt install -s nginx                 # Simulate (dry run)
$ dnf install --assumeno nginx         # Show but don't install
```

### Tip 3: Fix broken packages
```bash
# Debian/Ubuntu
$ sudo apt --fix-broken install
$ sudo dpkg --configure -a

# RHEL/Fedora
$ sudo dnf distro-sync
```

### Tip 4: Search for what you need
```bash
# "I need a PDF viewer"
$ apt search "pdf viewer"
$ dnf search "pdf viewer"
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Automatic dependency resolution | Need internet for repos |
| GPG signature verification | Version may be older than upstream |
| Easy bulk operations | Distro-specific (apt vs dnf) |
| Transaction history (dnf) | PPA/third-party repos = risk |
| Unattended upgrades | Large cache on disk |

---

## 📍 Where & When to Use

| Scenario | Debian/Ubuntu | RHEL/Fedora |
|----------|--------------|-------------|
| Install software | `apt install` | `dnf install` |
| Update system | `apt update && upgrade` | `dnf upgrade` |
| Search packages | `apt search` | `dnf search` |
| Remove software | `apt purge` | `dnf remove` |
| Find file owner | `dpkg -S file` | `dnf provides file` |
| Undo install | Check logs | `dnf history undo` |
| Auto-updates | `unattended-upgrades` | `dnf-automatic` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| `apt install` without `apt update` | Always update first! |
| Using `remove` instead of `purge` | `purge` removes config too |
| Forgetting `autoremove` | Run periodically to free space |
| Adding untrusted PPAs | Verify source before adding |
| Not holding critical packages | `apt-mark hold pkg` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Update package list and upgrade system
2. Install and remove a package
3. Search for a package by keyword

### 🟡 Intermediate
4. Install a specific version of a package
5. Hold a package to prevent upgrades
6. Add a third-party repository

### 🔴 Advanced
7. Set up unattended security upgrades
8. Verify package integrity with `debsums`/`rpm -Va`
9. Use `dnf history undo` to rollback an install

---

## 🧠 Cheat Sheet

```
═══ DEBIAN/UBUNTU (apt) ═══
sudo apt update                → Refresh index
sudo apt upgrade -y            → Upgrade all
sudo apt install pkg           → Install
sudo apt remove pkg            → Remove (keep config)
sudo apt purge pkg             → Remove + config
sudo apt autoremove            → Clean unused deps
apt search term                → Search
apt show pkg                   → Package info
apt list --installed           → List installed
sudo apt-mark hold pkg         → Prevent upgrade
sudo apt clean                 → Clear cache

═══ RHEL/FEDORA (dnf) ═══
sudo dnf check-update          → Check updates
sudo dnf upgrade -y            → Upgrade all
sudo dnf install pkg           → Install
sudo dnf remove pkg            → Remove
dnf search term                → Search
dnf info pkg                   → Package info
dnf list installed             → List installed
dnf provides /path/file        → Find owner
sudo dnf history               → Transaction log
sudo dnf history undo N        → Rollback!
sudo dnf clean all             → Clear cache
```

---

> **Previous**: [`lsblk` ←](../06_disk_storage/45_lsblk.md) | **Next**: [`dpkg/rpm` →](./47_dpkg_rpm.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
