# 🛠️ `dpkg` & `rpm` — Low-Level Package Tools | Linux Master Note

> **The engines under apt and dnf. `dpkg` and `rpm` handle individual package files directly — install, query, verify, and extract without repositories.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--package-internals)
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

### High-level vs Low-level:
```
apt/dnf   → Resolves dependencies → Downloads from repos → Calls dpkg/rpm
dpkg/rpm  → Installs/removes individual .deb/.rpm files directly
```

| Feature | `dpkg` | `rpm` |
|---------|--------|-------|
| Distro | Debian/Ubuntu | RHEL/Fedora |
| Format | `.deb` | `.rpm` |
| Dependency resolution | ❌ No | ❌ No |
| Package database | `/var/lib/dpkg/` | `/var/lib/rpm/` |
| Query installed | ✅ Yes | ✅ Yes |
| Verify files | ✅ `debsums` | ✅ `rpm -V` |

---

## 📖 Theory — Package Internals

### .deb package structure:
```
package.deb
├── debian-binary        # Format version ("2.0")
├── control.tar.gz       # Metadata
│   ├── control          # Name, version, depends
│   ├── preinst          # Pre-install script
│   ├── postinst         # Post-install script
│   ├── prerm            # Pre-remove script
│   └── postrm           # Post-remove script
└── data.tar.gz          # Actual files
    ├── usr/bin/
    ├── etc/
    └── lib/
```

### .rpm package structure:
```
package.rpm
├── Lead                 # RPM magic number
├── Signature            # GPG signature
├── Header               # Metadata (name, version, deps)
└── Payload              # Compressed cpio archive
    ├── usr/bin/
    ├── etc/
    └── lib/
```

---

## 🧰 Syntax & Options

### `dpkg` (Debian/Ubuntu):
| Command | Description |
|---------|-------------|
| `dpkg -i file.deb` | **Install** package |
| `dpkg -r package` | **Remove** package (keep config) |
| `dpkg -P package` | **Purge** package + config |
| `dpkg -l` | **List** all installed packages |
| `dpkg -l pattern` | List matching packages |
| `dpkg -L package` | List **files** in installed package |
| `dpkg -S file` | Find which package **owns** a file |
| `dpkg -s package` | **Status/info** of installed package |
| `dpkg -c file.deb` | List **contents** of .deb file |
| `dpkg -I file.deb` | Show **info** from .deb file |
| `dpkg --configure -a` | Fix broken installations |
| `dpkg --get-selections` | Export installed package list |
| `dpkg --set-selections` | Import package list |
| `dpkg -x file.deb dir/` | **Extract** .deb contents |

### `rpm` (RHEL/Fedora):
| Command | Description |
|---------|-------------|
| `rpm -ivh file.rpm` | **Install** (verbose + progress) |
| `rpm -Uvh file.rpm` | **Upgrade** (or install if new) |
| `rpm -e package` | **Remove** package |
| `rpm -qa` | List **all** installed packages |
| `rpm -qi package` | **Info** about installed package |
| `rpm -ql package` | List **files** in package |
| `rpm -qf file` | Find which package **owns** a file |
| `rpm -qp file.rpm` | Query **uninstalled** .rpm file |
| `rpm -qpi file.rpm` | Info from .rpm file |
| `rpm -qpl file.rpm` | List files in .rpm file |
| `rpm -Va` | **Verify** all installed packages |
| `rpm -V package` | Verify specific package |
| `rpm -K file.rpm` | Check **signature** |
| `rpm --import KEY` | Import GPG key |

---

## 🟢 Basic Usage

### Install from file
```bash
# Debian/Ubuntu
$ sudo dpkg -i google-chrome-stable.deb
# If dependency errors:
$ sudo apt --fix-broken install

# RHEL/Fedora
$ sudo rpm -ivh google-chrome-stable.rpm
# If dependency errors:
$ sudo dnf install ./google-chrome-stable.rpm   # Better — resolves deps!
```

### Remove package
```bash
# Debian/Ubuntu
$ sudo dpkg -r google-chrome-stable    # Keep config
$ sudo dpkg -P google-chrome-stable    # Purge everything

# RHEL/Fedora
$ sudo rpm -e google-chrome-stable
```

### List installed packages
```bash
# Debian/Ubuntu
$ dpkg -l                              # All packages
$ dpkg -l | grep nginx                 # Search
$ dpkg -l | grep "^ii"                 # Only installed

# RHEL/Fedora
$ rpm -qa                              # All packages
$ rpm -qa | grep nginx                 # Search
$ rpm -qa --last | head -20            # Recently installed
```

---

## 🟡 Intermediate Usage

### Find which package owns a file
```bash
# Debian/Ubuntu
$ dpkg -S /usr/bin/curl
curl: /usr/bin/curl

$ dpkg -S /etc/nginx/nginx.conf
nginx-common: /etc/nginx/nginx.conf

# RHEL/Fedora
$ rpm -qf /usr/bin/curl
curl-7.76.1-23.el9.x86_64

$ rpm -qf /etc/nginx/nginx.conf
nginx-1.22.1-1.el9.x86_64
```

### List files installed by a package
```bash
# Debian/Ubuntu
$ dpkg -L nginx
/usr/sbin/nginx
/etc/nginx/nginx.conf
/var/log/nginx/
/usr/share/doc/nginx/

# RHEL/Fedora
$ rpm -ql nginx
```

### Package information
```bash
# Debian/Ubuntu
$ dpkg -s nginx
Package: nginx
Status: install ok installed
Version: 1.22.0-1ubuntu1
Description: small, powerful web server

# From .deb file (before installing)
$ dpkg -I ./package.deb

# RHEL/Fedora
$ rpm -qi nginx
Name        : nginx
Version     : 1.22.1
Summary     : A high performance web server

# From .rpm file
$ rpm -qpi ./package.rpm
```

### Inspect package contents without installing
```bash
# Debian/Ubuntu — list contents of .deb
$ dpkg -c package.deb
drwxr-xr-x  usr/bin/
-rwxr-xr-x  usr/bin/myapp
drwxr-xr-x  etc/myapp/
-rw-r--r--  etc/myapp/config.yml

# Extract without installing
$ dpkg -x package.deb /tmp/extracted/

# RHEL/Fedora — list contents of .rpm
$ rpm -qpl package.rpm
$ rpm2cpio package.rpm | cpio -t       # List files

# Extract without installing
$ rpm2cpio package.rpm | cpio -idmv
```

---

## 🔴 Advanced Usage

### Package Verification (Security) 🔒
```bash
# Debian/Ubuntu — verify file integrity
$ sudo apt install debsums
$ sudo debsums -c                      # Check for modified files
$ sudo debsums -c nginx                # Check specific package
# Output shows files that differ from package version

# RHEL/Fedora — verify packages
$ rpm -Va                              # Verify ALL packages
$ rpm -V nginx                         # Verify specific package
# Output codes:
# S = Size    M = Mode      5 = MD5 checksum
# D = Device  L = Symlink   U = User    G = Group   T = mTime

# Example output:
S.5....T.  c /etc/nginx/nginx.conf     # Config was modified (normal)
..5....T.    /usr/bin/something         # Binary modified (SUSPICIOUS!)

# Check RPM signatures
$ rpm -K package.rpm
package.rpm: digests signatures OK
```

### Export/Import package list (system migration)
```bash
# Debian/Ubuntu — Export installed packages
$ dpkg --get-selections > installed_packages.txt

# Import on new system
$ sudo dpkg --set-selections < installed_packages.txt
$ sudo apt-get dselect-upgrade

# RHEL/Fedora
$ rpm -qa --qf '%{NAME}\n' > packages.txt
$ sudo dnf install $(cat packages.txt)
```

### Fix broken packages
```bash
# Debian/Ubuntu
$ sudo dpkg --configure -a             # Reconfigure all
$ sudo apt --fix-broken install        # Fix dependencies
$ sudo apt install -f                  # Same as above

# If dpkg database corrupted:
$ sudo cp /var/lib/dpkg/status-old /var/lib/dpkg/status
$ sudo apt update
```

### CTF/Pentesting — Package Forensics 🎯
```bash
# What was recently installed? (backdoor detection)
# Debian/Ubuntu
$ grep " install " /var/log/dpkg.log | tail -20
$ zgrep " install " /var/log/dpkg.log.*.gz   # Older logs

# RHEL/Fedora
$ rpm -qa --last | head -20

# Find modified system binaries (rootkit detection!)
$ sudo debsums -c 2>/dev/null | grep -v "OK$"
$ sudo rpm -Va 2>/dev/null | grep "^..5" | grep -v "^..5....T.  c"
# Modified non-config files = SUSPICIOUS!

# Check if package is genuine
$ dpkg -s suspicious-package
$ rpm -qi suspicious-package
# Check: Source, Maintainer, Signature
```

---

## 💡 Real World Pro Tips

### Tip 1: Use `dnf install ./file.rpm` instead of `rpm -i`
```bash
# ❌ rpm won't resolve dependencies
$ sudo rpm -ivh package.rpm
error: Failed dependencies: libfoo.so.1 is needed

# ✅ dnf resolves dependencies automatically
$ sudo dnf install ./package.rpm
```

### Tip 2: Reconfigure a package
```bash
# Debian/Ubuntu — re-run setup wizard
$ sudo dpkg-reconfigure tzdata         # Timezone
$ sudo dpkg-reconfigure locales        # Language
$ sudo dpkg-reconfigure keyboard-configuration
```

### Tip 3: List only manually installed packages
```bash
# Debian/Ubuntu
$ apt-mark showmanual

# RHEL/Fedora
$ dnf history userinstalled
```

### Tip 4: Find which package to install
```bash
# "I need the 'curl' command but it's not installed"
# Debian/Ubuntu
$ sudo apt install apt-file
$ sudo apt-file update
$ apt-file search /usr/bin/curl

# RHEL/Fedora
$ dnf provides "*/curl"
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Direct package file handling | No dependency resolution |
| Package verification | Must download .deb/.rpm manually |
| File ownership queries | More complex than apt/dnf |
| Extract without installing | Can break system if forced |
| System migration support | Different syntax (dpkg vs rpm) |

---

## 📍 Where & When to Use

| Scenario | Tool | Why |
|----------|------|-----|
| Install from file | `dpkg -i` / `rpm -ivh` | Direct install |
| Find file owner | `dpkg -S` / `rpm -qf` | Which package? |
| List package files | `dpkg -L` / `rpm -ql` | What was installed? |
| Verify integrity | `debsums` / `rpm -Va` | Security audit |
| Inspect without install | `dpkg -c` / `rpm -qpl` | Preview contents |
| Fix broken system | `dpkg --configure -a` | Recovery |
| System migration | `dpkg --get-selections` | Clone packages |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `rpm -i` instead of `dnf install ./` | dnf resolves deps |
| Ignoring dependency errors | `apt --fix-broken install` |
| Force-installing (`--force`) | Almost never the right fix |
| Not verifying signatures | `rpm -K` before install |
| Confusing package name with filename | `dpkg -r nginx` not `dpkg -r nginx.deb` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. List all installed packages with `dpkg -l` or `rpm -qa`
2. Find which package owns `/usr/bin/python3`
3. Show info about an installed package

### 🟡 Intermediate
4. Download a .deb/.rpm and inspect its contents
5. Extract a package without installing it
6. List all files installed by the `nginx` package

### 🔴 Advanced
7. Verify all installed packages for integrity
8. Export and import package lists between systems
9. Find modified system binaries (security audit)

---

## 🧠 Cheat Sheet

```
═══ DPKG (Debian/Ubuntu) ═══
dpkg -i file.deb          → Install
dpkg -r package           → Remove
dpkg -P package           → Purge
dpkg -l                   → List installed
dpkg -L package           → List package files
dpkg -S /path/file        → Find owner
dpkg -s package           → Package info
dpkg -c file.deb          → List .deb contents
dpkg -x file.deb dir/     → Extract
dpkg --configure -a       → Fix broken
debsums -c                → Verify integrity

═══ RPM (RHEL/Fedora) ═══
rpm -ivh file.rpm         → Install
rpm -Uvh file.rpm         → Upgrade
rpm -e package            → Remove
rpm -qa                   → List installed
rpm -ql package           → List package files
rpm -qf /path/file        → Find owner
rpm -qi package           → Package info
rpm -qpl file.rpm         → List .rpm contents
rpm -Va                   → Verify ALL
rpm -K file.rpm           → Check signature
```

---

> **Previous**: [`apt/yum/dnf` ←](./46_apt_yum_dnf.md) | **Next**: [`snap/flatpak` →](./48_snap_flatpak.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
