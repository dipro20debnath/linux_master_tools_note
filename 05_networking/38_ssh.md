# 🛠️ `ssh` — Secure Shell Remote Login | Linux Master Note

> **The backbone of remote Linux administration. `ssh` gives you encrypted, secure access to any machine anywhere in the world.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--ssh-protocol)
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

### What is SSH?
SSH (**S**ecure **Sh**ell) provides encrypted communication between two machines. It replaces insecure protocols like Telnet, rlogin, and rsh.

### SSH can do:
- **Remote shell** — Login to remote machines
- **File transfer** — SCP, SFTP
- **Port forwarding** — Tunnel traffic through SSH
- **X11 forwarding** — Run GUI apps remotely
- **Proxy/SOCKS** — Dynamic port forwarding
- **Key-based auth** — Passwordless, more secure

---

## 📖 Theory — SSH Protocol

### How SSH works:
```
1. Client connects to server (port 22)
2. Server sends its public host key
3. Client verifies key (known_hosts)
4. Key exchange (Diffie-Hellman) → shared secret
5. Symmetric encryption established
6. Authentication (password or key)
7. Encrypted session begins
```

### Authentication methods:
| Method | Security | Use Case |
|--------|----------|----------|
| **Password** | ⚠️ Moderate | Quick access |
| **Public key** | ✅ Strong | Daily use (RECOMMENDED) |
| **Certificate** | ✅ Strong | Enterprise/large scale |

### Key types:
| Type | Command | Security |
|------|---------|----------|
| RSA | `ssh-keygen -t rsa -b 4096` | ✅ Good (4096-bit) |
| Ed25519 | `ssh-keygen -t ed25519` | ✅ Best (modern, fast) |
| ECDSA | `ssh-keygen -t ecdsa` | ✅ Good |

### SSH files:
| File | Purpose |
|------|---------|
| `~/.ssh/id_rsa` | Your private key (KEEP SECRET!) |
| `~/.ssh/id_rsa.pub` | Your public key (share freely) |
| `~/.ssh/known_hosts` | Trusted server fingerprints |
| `~/.ssh/authorized_keys` | Keys allowed to login (on server) |
| `~/.ssh/config` | Client configuration |
| `/etc/ssh/sshd_config` | Server configuration |

---

## 🧰 Syntax & Options

```bash
ssh [OPTIONS] [user@]hostname [command]
```

| Flag | Description |
|------|-------------|
| `-p PORT` | Connect to specific port |
| `-i KEY` | Use specific private key |
| `-L local:host:remote` | Local port forwarding |
| `-R remote:host:local` | Remote port forwarding |
| `-D PORT` | Dynamic SOCKS proxy |
| `-N` | No command (just forwarding) |
| `-f` | Background after auth |
| `-v` / `-vv` / `-vvv` | Verbose (debug) |
| `-X` | Enable X11 forwarding |
| `-A` | Agent forwarding |
| `-J jumphost` | Jump through proxy host |
| `-o OPTION` | Set config option |
| `-t` | Force pseudo-terminal |
| `-C` | Enable compression |

---

## 🟢 Basic Usage

```bash
# Connect to remote server
$ ssh user@192.168.1.100
$ ssh dipro@server.example.com

# Connect on different port
$ ssh -p 2222 user@server.com

# Run single command remotely
$ ssh user@server "ls -la /var/www"
$ ssh user@server "df -h && free -h"

# Connect with specific key
$ ssh -i ~/.ssh/my_key user@server

# First connection — verify fingerprint
$ ssh user@server
The authenticity of host 'server' can't be established.
ED25519 key fingerprint is SHA256:abc123...
Are you sure you want to continue connecting (yes/no)? yes
```

---

## 🟡 Intermediate Usage

### Generate SSH keys
```bash
# Generate Ed25519 key (RECOMMENDED)
$ ssh-keygen -t ed25519 -C "dipro@example.com"
Generating public/private ed25519 key pair.
Enter file: ~/.ssh/id_ed25519
Enter passphrase: ********

# Generate RSA key (4096-bit)
$ ssh-keygen -t rsa -b 4096 -C "dipro@example.com"
```

### Copy key to server (passwordless login)
```bash
$ ssh-copy-id user@server
# OR manually:
$ cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### SSH config file (`~/.ssh/config`)
```bash
# ~/.ssh/config — makes life SO much easier!
Host myserver
    HostName 192.168.1.100
    User dipro
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

Host production
    HostName prod.example.com
    User deploy
    IdentityFile ~/.ssh/prod_key

Host htb
    HostName 10.10.10.100
    User root
    IdentityFile ~/.ssh/htb_key
    StrictHostKeyChecking no

# Now just:
$ ssh myserver       # Instead of: ssh -p 2222 -i key dipro@192.168.1.100
$ ssh production
```

### SSH agent (remember passphrase)
```bash
# Start agent
$ eval $(ssh-agent)

# Add key (enter passphrase once)
$ ssh-add ~/.ssh/id_ed25519

# List loaded keys
$ ssh-add -l

# Now SSH won't ask for passphrase again!
```

---

## 🔴 Advanced Usage

### Port Forwarding (Tunneling) 🔒

#### Local port forwarding (access remote service locally)
```bash
# Access remote MySQL (3306) through SSH
$ ssh -L 3307:localhost:3306 user@dbserver -N
# Now connect to localhost:3307 → reaches dbserver:3306

# Access remote web app
$ ssh -L 8080:localhost:80 user@webserver -N
# Open browser: http://localhost:8080

# Access service on internal network
$ ssh -L 9090:internal-server:8080 user@jumphost -N
# internal-server:8080 is now at localhost:9090
```

#### Remote port forwarding (expose local service remotely)
```bash
# Make your local web server accessible from remote
$ ssh -R 8080:localhost:3000 user@remote-server -N
# remote-server:8080 → your localhost:3000
```

#### Dynamic port forwarding (SOCKS proxy)
```bash
# Create SOCKS5 proxy
$ ssh -D 9050 user@server -N
# Configure browser proxy: SOCKS5 127.0.0.1:9050
# ALL traffic goes through the SSH tunnel!
```

### Jump hosts / Bastion
```bash
# Jump through bastion host
$ ssh -J bastion@jump.example.com user@internal-server

# In config:
Host internal
    HostName 10.0.0.5
    User admin
    ProxyJump bastion@jump.example.com
```

### SSH hardening (server-side) 🛡️
```bash
# /etc/ssh/sshd_config
Port 2222                          # Change default port
PermitRootLogin no                 # Disable root login
PasswordAuthentication no          # Key-only auth
PubkeyAuthentication yes
MaxAuthTries 3
AllowUsers dipro admin             # Whitelist users
LoginGraceTime 30                  # 30 seconds to auth
ClientAliveInterval 300            # Timeout idle sessions
ClientAliveCountMax 2
X11Forwarding no                   # Disable unless needed
AllowTcpForwarding no              # Disable unless needed
Protocol 2                         # SSH v2 only

# Restart sshd
$ sudo systemctl restart sshd
```

### CTF/Pentesting — SSH Exploitation 🎯
```bash
# SSH with found credentials
$ ssh user@target -p 22

# SSH with private key (found during enumeration)
$ chmod 600 found_key
$ ssh -i found_key user@target

# SSH tunneling to access internal services
$ ssh -L 8080:127.0.0.1:8080 user@target -N
# Now browse internal web app at localhost:8080

# SSH to pivot to another network
$ ssh -D 1080 user@compromised -N
$ proxychains nmap -sT 10.0.0.0/24

# Brute force SSH (use responsibly!)
$ hydra -l admin -P wordlist.txt ssh://target
```

---

## 💡 Real World Pro Tips

### Tip 1: Use SSH config — stop typing long commands!
```bash
# Instead of: ssh -p 2222 -i ~/.ssh/key dipro@192.168.1.100
# Just: ssh myserver
```

### Tip 2: Keep connections alive
```bash
# In ~/.ssh/config:
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

### Tip 3: Copy files through SSH quickly
```bash
$ ssh user@server "cat /remote/file" > local_file
$ cat local_file | ssh user@server "cat > /remote/file"
```

### Tip 4: Run commands on multiple servers
```bash
$ for server in server1 server2 server3; do
    echo "=== $server ==="
    ssh "$server" "uptime && df -h /"
done
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Encrypted communication | Complex tunnel syntax |
| Key-based authentication | Key management overhead |
| Port forwarding/tunneling | Can be slow over high latency |
| Universal (every server) | Misconfiguration = security risk |
| X11 and agent forwarding | Brute-forceable if password auth on |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Remote server access | `ssh user@host` | Encrypted shell |
| Passwordless login | SSH keys | Automation |
| Access internal service | Local port forward | Tunnel through |
| Anonymous browsing | SOCKS proxy | Privacy |
| CTF pivoting | Dynamic forward | Access internal nets |
| Server hardening | sshd_config | Security |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Key permissions too open | `chmod 600 ~/.ssh/id_*` |
| Root login enabled | `PermitRootLogin no` |
| Password auth on public server | `PasswordAuthentication no` |
| Not verifying host fingerprint | Check on first connect |
| SSH on default port 22 | Change to non-standard port |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Generate an Ed25519 SSH key pair
2. SSH into a remote server
3. Run a command remotely without interactive shell

### 🟡 Intermediate
4. Set up passwordless SSH with key auth
5. Create SSH config for 3 servers
6. Set up local port forwarding

### 🔴 Advanced
7. Configure SSH hardening on a server
8. Set up a SOCKS proxy for pivoting
9. Use jump hosts to access internal servers

---

## 🧠 Cheat Sheet

```
CONNECT:
  ssh user@host                → Basic login
  ssh -p 2222 user@host        → Custom port
  ssh -i key user@host         → Specific key

KEYS:
  ssh-keygen -t ed25519        → Generate key
  ssh-copy-id user@host        → Deploy key
  ssh-add ~/.ssh/key           → Load in agent

TUNNEL:
  ssh -L 8080:host:80 user@gw -N    → Local forward
  ssh -R 8080:localhost:80 user@srv  → Remote forward
  ssh -D 9050 user@host -N          → SOCKS proxy

JUMP:
  ssh -J user@bastion user@internal

CONFIG (~/.ssh/config):
  Host name
      HostName IP
      User username
      Port 2222
      IdentityFile ~/.ssh/key

HARDENING:
  PermitRootLogin no
  PasswordAuthentication no
  Port 2222
  AllowUsers user1 user2
```

---

> **Previous**: [`wget` ←](./37_wget.md) | **Next**: [`scp/rsync` →](./39_scp_rsync.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
