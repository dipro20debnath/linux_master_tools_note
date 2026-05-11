# 🛠️ `ifconfig` & `ip` — Configure Network Interfaces | Linux Master Note

> **See and configure your network interfaces. `ifconfig` is the classic, `ip` is the modern replacement. Every sysadmin and hacker must know both.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--network-interfaces)
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

### `ifconfig` vs `ip`:
| Feature | `ifconfig` | `ip` |
|---------|-----------|------|
| Status | ⚠️ **Deprecated** (net-tools) | ✅ **Modern** (iproute2) |
| Package | `net-tools` | `iproute2` (pre-installed) |
| Capability | Basic interface config | Full networking control |
| VLAN/Tunnel | ❌ No | ✅ Yes |
| Namespace | ❌ No | ✅ Yes |
| Output format | Human-friendly | JSON available |

> 🎯 **Learn `ip` — it's the future.** But know `ifconfig` — you'll see it everywhere in older docs and CTFs.

---

## 📖 Theory — Network Interfaces

### Common interface names:
| Name | Type | Description |
|------|------|-------------|
| `lo` | Loopback | Localhost (127.0.0.1) |
| `eth0` | Ethernet | Wired connection (old naming) |
| `ens33` | Ethernet | Wired (predictable naming) |
| `wlan0` | WiFi | Wireless interface |
| `wlp2s0` | WiFi | Wireless (predictable naming) |
| `docker0` | Bridge | Docker network bridge |
| `tun0` | Tunnel | VPN tunnel (OpenVPN, HTB) |
| `virbr0` | Bridge | Virtual machine bridge |

### Key network attributes:
| Attribute | Description |
|-----------|-------------|
| **IP Address** | Machine's network address |
| **Netmask** | Defines network size (255.255.255.0 = /24) |
| **Broadcast** | Address to reach all hosts on subnet |
| **MAC Address** | Hardware address (unique per NIC) |
| **MTU** | Maximum Transmission Unit (default: 1500) |
| **Flags** | UP, BROADCAST, RUNNING, MULTICAST |

---

## 🧰 Syntax & Options

### `ifconfig`:
```bash
ifconfig [INTERFACE] [OPTIONS]
```

### `ip` subcommands:
```bash
ip [OPTIONS] OBJECT COMMAND
```

| Object | Purpose | Example |
|--------|---------|---------|
| `addr` / `a` | IP addresses | `ip addr show` |
| `link` / `l` | Network interfaces | `ip link show` |
| `route` / `r` | Routing table | `ip route show` |
| `neigh` / `n` | ARP table | `ip neigh show` |
| `netns` | Network namespaces | `ip netns list` |

| `ip` Option | Description |
|-------------|-------------|
| `-4` | IPv4 only |
| `-6` | IPv6 only |
| `-c` | Color output |
| `-br` | Brief/short output |
| `-j` | JSON output |
| `-s` | Statistics |

---

## 🟢 Basic Usage

### View all interfaces
```bash
# ifconfig (classic)
$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
        ether 00:0c:29:ab:cd:ef  txqueuelen 1000
        RX bytes:123456  TX bytes:789012

# ip (modern — RECOMMENDED)
$ ip addr show
$ ip a                    # Short form
2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
    link/ether 00:0c:29:ab:cd:ef

# Brief output (clean!)
$ ip -br addr
lo       UNKNOWN  127.0.0.1/8
eth0     UP       192.168.1.100/24
wlan0    DOWN

# Color output
$ ip -c addr
```

### View specific interface
```bash
$ ifconfig eth0
$ ip addr show eth0
```

### View just IP addresses
```bash
# Quick — just IPs
$ hostname -I
192.168.1.100 10.0.0.1

# From ip command
$ ip -4 addr show | grep inet | awk '{print $2}'
```

### View routing table
```bash
# Classic
$ route -n

# Modern
$ ip route show
$ ip r
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100
```

### View ARP table
```bash
# Classic
$ arp -a

# Modern
$ ip neigh show
192.168.1.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

---

## 🟡 Intermediate Usage

### Assign IP address
```bash
# ifconfig
$ sudo ifconfig eth0 192.168.1.200 netmask 255.255.255.0 up

# ip (modern)
$ sudo ip addr add 192.168.1.200/24 dev eth0
$ sudo ip link set eth0 up
```

### Remove IP address
```bash
$ sudo ip addr del 192.168.1.200/24 dev eth0
```

### Bring interface up/down
```bash
# ifconfig
$ sudo ifconfig eth0 down
$ sudo ifconfig eth0 up

# ip
$ sudo ip link set eth0 down
$ sudo ip link set eth0 up
```

### Add/change default gateway
```bash
# Add default route
$ sudo ip route add default via 192.168.1.1

# Delete default route
$ sudo ip route del default

# Change default route
$ sudo ip route replace default via 10.0.0.1 dev eth0
```

### Change MAC address (spoofing) 🔒
```bash
# Bring interface down first
$ sudo ip link set eth0 down

# Change MAC
$ sudo ip link set eth0 address aa:bb:cc:dd:ee:ff

# Bring back up
$ sudo ip link set eth0 up

# Or use macchanger tool
$ sudo macchanger -r eth0      # Random MAC
$ sudo macchanger -m aa:bb:cc:dd:ee:ff eth0
```

### View interface statistics
```bash
$ ip -s link show eth0
    RX: bytes  packets  errors  dropped
    12345678   98765    0       0
    TX: bytes  packets  errors  dropped
    87654321   76543    0       0
```

---

## 🔴 Advanced Usage

### Multiple IPs on one interface
```bash
# Add secondary IP (virtual interface)
$ sudo ip addr add 192.168.1.201/24 dev eth0 label eth0:1
$ sudo ip addr add 10.0.0.100/24 dev eth0 label eth0:2

# List all IPs
$ ip addr show eth0
```

### VLAN configuration
```bash
# Create VLAN interface
$ sudo ip link add link eth0 name eth0.100 type vlan id 100
$ sudo ip addr add 10.10.100.1/24 dev eth0.100
$ sudo ip link set eth0.100 up
```

### Network namespaces (container networking)
```bash
# Create namespace
$ sudo ip netns add test_ns

# Run command in namespace
$ sudo ip netns exec test_ns ip addr show

# Connect namespace to host via veth pair
$ sudo ip link add veth0 type veth peer name veth1
$ sudo ip link set veth1 netns test_ns
```

### Pentesting — Network Reconnaissance 🎯
```bash
# What's my IP on the VPN? (TryHackMe/HTB)
$ ip addr show tun0 | grep inet | awk '{print $2}'
10.10.14.25/23

# Find your network interfaces
$ ip -br link
lo       UNKNOWN  00:00:00:00:00:00
eth0     UP       00:0c:29:ab:cd:ef
tun0     UNKNOWN  <pointopoint>

# Check routing (where does traffic go?)
$ ip route
default via 192.168.1.1 dev eth0
10.10.10.0/23 dev tun0 scope link

# ARP scan (find hosts on local network)
$ ip neigh show
$ arp -a
```

---

## 💡 Real World Pro Tips

### Tip 1: Get your IP fast
```bash
# Local IP
$ hostname -I | awk '{print $1}'

# Public IP
$ curl -s ifconfig.me
$ curl -s ipinfo.io/ip
```

### Tip 2: `ip -br a` is the best quick view
```bash
$ ip -br a
lo       UNKNOWN  127.0.0.1/8
eth0     UP       192.168.1.100/24
tun0     UNKNOWN  10.10.14.25/23
```

### Tip 3: Persistent network config
```bash
# ip/ifconfig changes are TEMPORARY — lost on reboot!
# For permanent config:
# Ubuntu: /etc/netplan/*.yaml
# Debian: /etc/network/interfaces
# RHEL: /etc/sysconfig/network-scripts/
```

### Tip 4: Flush all IPs
```bash
$ sudo ip addr flush dev eth0      # Remove all IPs from interface
```

---

## ✅ Pros & Cons

| ✅ `ip` Pros | ❌ `ifconfig` Cons |
|-------------|-------------------|
| Modern, actively maintained | Deprecated, may not be installed |
| JSON output for scripting | Limited functionality |
| VLAN, namespace support | No namespace support |
| Consistent syntax | Inconsistent between distros |
| Color and brief modes | No brief mode |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Quick IP check | `ip -br a` | Clean output |
| Full interface details | `ip addr show` | Complete info |
| Set static IP | `ip addr add` | Modern standard |
| Check routing | `ip route` | See traffic paths |
| MAC spoofing | `ip link set address` | Pentesting |
| VPN IP (HTB/THM) | `ip a show tun0` | Target network |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `ifconfig` on modern systems | Use `ip` (iproute2) |
| Config lost after reboot | Edit netplan/interfaces files |
| Forgetting to bring interface up | `ip link set dev up` |
| Not checking routing table | `ip route` before network changes |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. View all network interfaces with `ip -br a`
2. Find your default gateway with `ip route`
3. Check your MAC address

### 🟡 Intermediate
4. Assign a secondary IP to your interface
5. Change your MAC address and verify
6. View ARP table and interface statistics

### 🔴 Advanced
7. Create a network namespace and connect it
8. Set up a VLAN interface
9. Write a script to display all network info

---

## 🧠 Cheat Sheet

```
VIEW:
  ip -br a              → Quick interface list
  ip addr show          → Detailed addresses
  ip link show          → Interface status
  ip route              → Routing table
  ip neigh              → ARP table
  ip -s link show eth0  → Statistics

CONFIGURE:
  sudo ip addr add IP/MASK dev IFACE   → Add IP
  sudo ip addr del IP/MASK dev IFACE   → Remove IP
  sudo ip link set IFACE up/down       → Enable/disable
  sudo ip route add default via GW     → Set gateway
  sudo ip link set IFACE address MAC   → Change MAC

QUICK:
  hostname -I           → All local IPs
  curl ifconfig.me      → Public IP
  ip a show tun0        → VPN IP

LEGACY (ifconfig):
  ifconfig              → All interfaces
  ifconfig eth0 IP up   → Set IP
```

---

> **Previous**: [`ping` ←](./33_ping.md) | **Next**: [`netstat/ss` →](./35_netstat_ss.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
