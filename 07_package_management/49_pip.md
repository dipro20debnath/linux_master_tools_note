# 🛠️ `pip` / `pip3` — Python Package Manager | Linux Master Note

> **Install Python libraries with a single command. `pip` manages the entire Python ecosystem — from data science to web frameworks to pentesting tools.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--python-packaging)
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

### What is pip?
`pip` (**P**ip **I**nstalls **P**ackages) is the package installer for Python. It downloads and installs packages from **PyPI** (Python Package Index — pypi.org), which hosts 400,000+ packages.

### `pip` vs `pip3`:
| Command | Python Version | Notes |
|---------|---------------|-------|
| `pip` | Python 2 (legacy) | ⚠️ Deprecated |
| `pip3` | Python 3 | ✅ Use this! |
| `python3 -m pip` | Python 3 | ✅ **Most reliable** way |

> 🎯 Always use `python3 -m pip` to ensure correct Python version.

---

## 📖 Theory — Python Packaging

### Package sources:
```
PyPI (pypi.org) ← pip downloads from here
     │
     ▼
pip install package
     │
     ├── Downloads wheel (.whl) or source (.tar.gz)
     ├── Resolves dependencies
     ├── Installs to site-packages/
     │   ├── /usr/lib/python3/dist-packages/  (system)
     │   ├── ~/.local/lib/python3.x/site-packages/  (user)
     │   └── venv/lib/python3.x/site-packages/  (virtual env)
     └── Installs CLI scripts to bin/
```

### Virtual Environments (CRITICAL!):
```
System Python        Virtual Environment
/usr/bin/python3     /home/dipro/project/venv/bin/python3
     │                    │
     │ Global packages    │ Isolated packages
     │ (shared, risky)    │ (per-project, safe)
     │                    │
     └── DON'T install    └── ALWAYS install here!
         packages here
```

> ⚠️ **NEVER use `sudo pip install`** on modern systems. Use virtual environments!

---

## 🧰 Syntax & Options

```bash
pip3 [COMMAND] [OPTIONS] [PACKAGES...]
```

| Command | Description |
|---------|-------------|
| `install PKG` | Install package(s) |
| `uninstall PKG` | Remove package(s) |
| `list` | List installed packages |
| `show PKG` | Package details |
| `search TERM` | Search PyPI (deprecated — use web) |
| `freeze` | Output installed in requirements format |
| `download PKG` | Download without installing |
| `check` | Verify dependency compatibility |
| `config` | Manage pip configuration |
| `cache` | Manage pip cache |
| `install -r FILE` | Install from requirements file |

| Option | Description |
|--------|-------------|
| `--user` | Install for current user only |
| `--upgrade` / `-U` | Upgrade to latest version |
| `--force-reinstall` | Reinstall even if up-to-date |
| `--no-deps` | Don't install dependencies |
| `--pre` | Include pre-release versions |
| `--index-url URL` | Custom PyPI mirror |
| `--no-cache-dir` | Don't use cache |
| `-e PATH` | Install in editable/development mode |
| `--target DIR` | Install to specific directory |
| `--require-virtualenv` | Only install in venv |

---

## 🟢 Basic Usage

### Install packages
```bash
# Install a package
$ pip3 install requests
$ pip3 install flask django numpy

# Install specific version
$ pip3 install requests==2.28.0
$ pip3 install "requests>=2.25,<3.0"

# Upgrade package
$ pip3 install --upgrade requests
$ pip3 install -U pip               # Upgrade pip itself

# Install for user only (no sudo!)
$ pip3 install --user package
```

### Remove packages
```bash
$ pip3 uninstall requests
$ pip3 uninstall -y requests flask   # Auto-yes
```

### List and show
```bash
# List installed
$ pip3 list
Package    Version
---------- -------
Flask      3.0.0
requests   2.31.0
numpy      1.26.0

# Outdated packages
$ pip3 list --outdated
Package  Version Latest Type
-------- ------- ------ -----
Flask    2.3.0   3.0.0  wheel
numpy    1.25.0  1.26.0 wheel

# Package details
$ pip3 show requests
Name: requests
Version: 2.31.0
Summary: Python HTTP for Humans
Location: /home/dipro/.local/lib/python3.11/site-packages
Requires: charset-normalizer, idna, urllib3, certifi
```

---

## 🟡 Intermediate Usage

### Virtual Environments (ESSENTIAL!)
```bash
# Create virtual environment
$ python3 -m venv myproject_env

# Activate it
$ source myproject_env/bin/activate     # Linux/macOS
(myproject_env) $                       # Prompt changes!

# Now install freely — isolated!
(myproject_env) $ pip install flask requests numpy

# Deactivate when done
(myproject_env) $ deactivate
$
```

### Requirements files
```bash
# Export current packages
$ pip3 freeze > requirements.txt

# requirements.txt contents:
Flask==3.0.0
requests==2.31.0
numpy==1.26.0
Jinja2==3.1.2

# Install from requirements
$ pip3 install -r requirements.txt

# Upgrade all from requirements
$ pip3 install -r requirements.txt --upgrade
```

### Install from various sources
```bash
# From PyPI (default)
$ pip3 install requests

# From GitHub
$ pip3 install git+https://github.com/user/repo.git
$ pip3 install git+https://github.com/user/repo.git@branch

# From local file
$ pip3 install ./package.whl
$ pip3 install ./package.tar.gz

# From local directory (editable/dev mode)
$ pip3 install -e ./myproject/

# From specific index
$ pip3 install --index-url https://pypi.company.com/simple/ package
```

### Check dependencies
```bash
$ pip3 check
# Shows any dependency conflicts
No broken requirements found.
```

### Cache management
```bash
# View cache info
$ pip3 cache info

# Clear cache
$ pip3 cache purge

# Install without cache
$ pip3 install --no-cache-dir package
```

---

## 🔴 Advanced Usage

### Project Setup with venv
```bash
#!/bin/bash
# project_setup.sh
PROJECT_NAME=$1

mkdir -p "$PROJECT_NAME"
cd "$PROJECT_NAME"

# Create venv
python3 -m venv venv

# Activate
source venv/bin/activate

# Install dependencies
pip install --upgrade pip
pip install flask requests pytest black

# Save requirements
pip freeze > requirements.txt

# Create project structure
mkdir -p src tests
touch src/__init__.py src/app.py
touch tests/__init__.py tests/test_app.py
touch .gitignore README.md

# .gitignore
cat > .gitignore << 'EOF'
venv/
__pycache__/
*.pyc
.env
*.egg-info/
dist/
build/
EOF

echo "Project $PROJECT_NAME created!"
echo "Activate with: source venv/bin/activate"
```

### Pentesting Python tools 🎯
```bash
# Create pentesting venv
$ python3 -m venv ~/pentest-env
$ source ~/pentest-env/bin/activate

# Install essential pentesting tools
$ pip install impacket           # Windows protocol attacks
$ pip install pwntools           # CTF exploitation
$ pip install scapy              # Packet manipulation
$ pip install sqlmap             # SQL injection
$ pip install requests           # HTTP library
$ pip install beautifulsoup4     # Web scraping
$ pip install paramiko           # SSH library
$ pip install cryptography       # Crypto toolkit
$ pip install python-nmap        # Nmap integration
$ pip install bloodhound         # AD enumeration

# Save your pentesting toolkit
$ pip freeze > ~/pentest-requirements.txt
```

### Security — Vulnerability Scanning 🔒
```bash
# Check for known vulnerabilities in packages
$ pip install pip-audit
$ pip-audit
Found 2 known vulnerabilities:
  requests 2.25.0 → CVE-2023-xxxxx (upgrade to 2.31.0)
  urllib3 1.26.0 → CVE-2023-yyyyy (upgrade to 2.0.0)

# Auto-fix
$ pip-audit --fix

# Check with safety
$ pip install safety
$ safety check
$ pip freeze | safety check --stdin
```

### Private PyPI mirror
```bash
# Use company mirror
$ pip3 install --index-url https://pypi.company.com/simple/ package

# Or in pip.conf
$ cat ~/.pip/pip.conf
[global]
index-url = https://pypi.company.com/simple/
trusted-host = pypi.company.com
```

### Upgrade all outdated packages
```bash
# List and upgrade all
$ pip3 list --outdated --format=json | python3 -c "
import json, sys
for pkg in json.load(sys.stdin):
    print(pkg['name'])" | xargs -n1 pip3 install -U
```

---

## 💡 Real World Pro Tips

### Tip 1: ALWAYS use virtual environments!
```bash
# ❌ NEVER
$ sudo pip3 install package    # Breaks system Python!

# ✅ ALWAYS
$ python3 -m venv venv
$ source venv/bin/activate
$ pip install package
```

### Tip 2: Pin versions in production
```bash
# ❌ Bad (unpredictable)
requests
flask

# ✅ Good (reproducible)
requests==2.31.0
flask==3.0.0
```

### Tip 3: Use `python3 -m pip` for reliability
```bash
# This ensures pip matches the Python version:
$ python3 -m pip install requests
# Instead of:
$ pip3 install requests    # Might use wrong Python!
```

### Tip 4: pipx for CLI tools
```bash
# Install Python CLI tools in isolated environments
$ pip3 install --user pipx
$ pipx install black
$ pipx install httpie
$ pipx install youtube-dl
# Each tool gets its own venv — clean!
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| 400,000+ packages on PyPI | Dependency conflicts ("pip hell") |
| Simple install syntax | No built-in environment isolation |
| Requirements files | Can break system Python |
| Git/URL install support | Package quality varies |
| Editable dev mode | No rollback/undo |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| New project | `python3 -m venv` + `pip` | Isolated setup |
| Install library | `pip install pkg` | PyPI access |
| Share dependencies | `pip freeze > requirements.txt` | Reproducibility |
| Pentesting tools | `pip install impacket` | Python security tools |
| Audit for vulns | `pip-audit` | Security |
| CLI tools | `pipx install tool` | Clean isolation |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| `sudo pip install` | Use venv instead! |
| Not pinning versions | `pip freeze > requirements.txt` |
| Installing globally | Use `--user` or venv |
| Ignoring venv | Every project should have one |
| Not auditing packages | Use `pip-audit` or `safety` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Install `requests` with pip and test it
2. List all installed packages
3. Show details of a specific package

### 🟡 Intermediate
4. Create a virtual environment and install packages
5. Generate and use a `requirements.txt` file
6. Install a package from GitHub

### 🔴 Advanced
7. Set up a complete project with venv and requirements
8. Audit installed packages for vulnerabilities
9. Create a pentesting Python toolkit

---

## 🧠 Cheat Sheet

```
INSTALL:
  pip3 install pkg                → Install
  pip3 install pkg==1.0.0         → Specific version
  pip3 install -U pkg             → Upgrade
  pip3 install -r requirements.txt → From file
  pip3 install --user pkg         → User-level

MANAGE:
  pip3 uninstall pkg              → Remove
  pip3 list                       → List installed
  pip3 list --outdated            → Show upgradable
  pip3 show pkg                   → Package info
  pip3 freeze > requirements.txt  → Export deps
  pip3 check                      → Verify deps

VIRTUAL ENVIRONMENT:
  python3 -m venv venv            → Create
  source venv/bin/activate        → Activate
  deactivate                      → Deactivate
  pip install pkg                 → Install (isolated!)

SECURITY:
  pip-audit                       → Check vulnerabilities
  safety check                    → Alternative audit

BEST PRACTICE:
  python3 -m pip install pkg      → Most reliable syntax
  NEVER sudo pip install          → Use venv instead!
```

---

> **Previous**: [`snap/flatpak` ←](./48_snap_flatpak.md) | **Next**: [`uname` →](../08_system_info/50_uname.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
