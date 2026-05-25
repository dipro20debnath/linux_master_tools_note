# 🛠️ `lscpu` & `lshw` — Hardware Information | Linux Master Note

> **Know your hardware inside out. `lscpu` reveals CPU details while `lshw` gives a full hardware inventory — essential for optimization, security, and forensics.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--hardware-information)
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

### Hardware info tools:
| Tool | Scope | Data Source |
|------|-------|-------------|
| `lscpu` | CPU only | `/proc/cpuinfo`, sysfs |
| `lshw` | **Full hardware** | DMI/SMBIOS, sysfs, /proc |
| `lspci` | PCI devices | PCI bus |
| `lsusb` | USB devices | USB bus |
| `dmidecode` | BIOS/firmware | DMI tables |
| `hwinfo` | Full (openSUSE) | Multiple sources |

---

## 📖 Theory — Hardware Information

### CPU architecture concepts:
| Term | Meaning |
|------|---------|
| **Socket** | Physical CPU chip on motherboard |
| **Core** | Independent processing unit within CPU |
| **Thread** | Virtual core (Hyper-Threading) |
| **Cache L1/L2/L3** | CPU memory hierarchy (fastest → slowest) |
| **Architecture** | x86_64, aarch64, etc. |
| **Byte Order** | Little Endian (Intel) / Big Endian |

### Relationship:
```
Socket(s) × Core(s) per socket × Thread(s) per core = Total CPUs
    1     ×        8           ×         2           =    16
```

### CPU vulnerabilities:
| Vulnerability | Impact | Kernel Check |
|--------------|--------|-------------|
| Spectre v1 | Bounds check bypass | `/sys/devices/system/cpu/vulnerabilities/spectre_v1` |
| Spectre v2 | Branch target injection | `/sys/devices/system/cpu/vulnerabilities/spectre_v2` |
| Meltdown | Rogue data cache load | `/sys/devices/system/cpu/vulnerabilities/meltdown` |

---

## 🧰 Syntax & Options

### `lscpu`:
```bash
lscpu [OPTIONS]
```

| Flag | Description |
|------|-------------|
| `-e` / `--extended` | Extended table format |
| `-p` | Parsable output |
| `-J` | **JSON** output |
| `-B` | Show in bytes |
| `-C` | Show cache info |
| `--all` | Include offline CPUs |

### `lshw`:
```bash
lshw [OPTIONS]
```

| Flag | Description |
|------|-------------|
| `-short` | **Compact** summary table |
| `-class CLASS` | Filter by class (cpu, memory, disk, network) |
| `-html` | HTML output |
| `-xml` | XML output |
| `-json` | JSON output |
| `-businfo` | Show bus information |
| `-sanitize` | Remove sensitive info |
| `-numeric` | Show numeric IDs |
| `-quiet` | Suppress warnings |

### lshw classes:
| Class | Description |
|-------|-------------|
| `cpu` / `processor` | CPU info |
| `memory` | RAM modules |
| `disk` | Storage devices |
| `network` | Network interfaces |
| `display` | GPU/graphics |
| `multimedia` | Audio |
| `bridge` | Bus bridges |
| `storage` | Storage controllers |

---

## 🟢 Basic Usage

### lscpu
```bash
$ lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                16
On-line CPU(s) list:   0-15
Thread(s) per core:    2
Core(s) per socket:    8
Socket(s):             1
Model name:            Intel(R) Core(TM) i7-10700K @ 3.80GHz
CPU MHz:               3800.000
CPU max MHz:           5100.0000
L1d cache:             256 KiB
L1i cache:             256 KiB
L2 cache:              2 MiB
L3 cache:              16 MiB
Virtualization:        VT-x
Vulnerability Spectre v1: Mitigation; usercopy/swapgs barriers
Vulnerability Spectre v2: Mitigation; Enhanced IBRS
Vulnerability Meltdown:   Not affected
```

### lshw
```bash
# Compact hardware summary (MOST USEFUL)
$ sudo lshw -short
H/W path        Device     Class          Description
=====================================================
                           system         OptiPlex 7080
/0                         bus            0J37VM
/0/0                       memory         64KiB BIOS
/0/4                       processor      Intel Core i7-10700K
/0/8                       memory         32GiB System Memory
/0/8/0                     memory         16GiB DIMM DDR4 3200 MHz
/0/8/1                     memory         16GiB DIMM DDR4 3200 MHz
/0/100/1f.2                storage        SATA Controller
/0/1           /dev/sda    disk           500GB WDC WD5000
/0/2           /dev/nvme0  disk           1TB Samsung SSD 970 EVO
/0/100/1f.6    enp0s31f6   network        Ethernet Connection
```

---

## 🟡 Intermediate Usage

### CPU details
```bash
# Extended CPU table
$ lscpu -e
CPU SOCKET CORE ONLINE MAXMHZ   MINMHZ
0   0      0    yes    5100.00  800.00
1   0      1    yes    5100.00  800.00
...

# Cache hierarchy
$ lscpu -C
NAME ONE-SIZE ALL-SIZE WAYS TYPE    LEVEL
L1d  32K      256K     8   Data    1
L1i  32K      256K     8   Instruction 1
L2   256K     2M       4   Unified 2
L3   16M      16M      16  Unified 3

# JSON output (scripting)
$ lscpu -J | jq '.lscpu[] | select(.field == "Model name:") | .data'
"Intel(R) Core(TM) i7-10700K @ 3.80GHz"
```

### Filter lshw by class
```bash
# CPU only
$ sudo lshw -class cpu
*-cpu
     product: Intel(R) Core(TM) i7-10700K @ 3.80GHz
     vendor: Intel Corp.
     width: 64 bits
     clock: 3800MHz
     capabilities: x86-64 vmx sse4_2 aes avx2

# Memory only
$ sudo lshw -class memory -short

# Network interfaces
$ sudo lshw -class network
*-network
     description: Ethernet interface
     product: Ethernet Connection (10) I219-LM
     vendor: Intel Corporation
     logical name: enp0s31f6
     serial: aa:bb:cc:dd:ee:ff
     capacity: 1Gbit/s
     width: 64 bits

# Disk info
$ sudo lshw -class disk

# GPU info
$ sudo lshw -class display
```

### /proc/cpuinfo deep dive
```bash
# Full CPU info
$ cat /proc/cpuinfo

# Quick summary
$ grep "model name" /proc/cpuinfo | head -1
$ grep "cpu cores" /proc/cpuinfo | head -1
$ grep "siblings" /proc/cpuinfo | head -1

# CPU flags (capabilities)
$ grep "flags" /proc/cpuinfo | head -1 | tr ' ' '\n' | sort
# Look for: aes, avx, avx2, vmx/svm (virtualization), sse4_2

# Check virtualization support
$ grep -E "vmx|svm" /proc/cpuinfo
# vmx = Intel VT-x, svm = AMD-V
```

### Related tools
```bash
# PCI devices (GPU, network, storage controllers)
$ lspci
00:02.0 VGA compatible controller: Intel UHD Graphics 630
00:1f.2 SATA controller: Intel Corporation
01:00.0 Network controller: Intel Wi-Fi 6 AX200

# Detailed PCI
$ lspci -v
$ lspci -nn    # With vendor/device IDs

# USB devices
$ lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 046d:c52b Logitech USB Receiver
Bus 001 Device 003: ID 8087:0029 Intel Corp. AX200 Bluetooth

# Detailed USB
$ lsusb -v -d 046d:c52b
```

---

## 🔴 Advanced Usage

### BIOS/Firmware info (dmidecode)
```bash
# System manufacturer and model
$ sudo dmidecode -t system
System Information
    Manufacturer: Dell Inc.
    Product Name: OptiPlex 7080
    Serial Number: ABC1234

# BIOS version
$ sudo dmidecode -t bios
BIOS Information
    Vendor: Dell Inc.
    Version: 1.15.0
    Release Date: 01/15/2024

# Memory modules
$ sudo dmidecode -t memory
Memory Device
    Size: 16 GB
    Type: DDR4
    Speed: 3200 MT/s
    Manufacturer: Samsung

# All serial numbers
$ sudo dmidecode -t system | grep Serial
$ sudo dmidecode -t baseboard | grep Serial
```

### CPU Vulnerability Check 🔒
```bash
# Check all CPU vulnerabilities
$ grep -r . /sys/devices/system/cpu/vulnerabilities/
spectre_v1:Mitigation; usercopy/swapgs barriers
spectre_v2:Mitigation; Enhanced IBRS
meltdown:Not affected
tsx_async_abort:Not affected
l1tf:Not affected
mds:Not affected

# Quick vulnerability summary
$ for v in /sys/devices/system/cpu/vulnerabilities/*; do
    echo "$(basename $v): $(cat $v)"
done
```

### Virtualization detection (CTF!) 🎯
```bash
# Are we in a VM?
$ lscpu | grep "Hypervisor vendor"
Hypervisor vendor: KVM        # ← We're in a VM!

# systemd approach
$ systemd-detect-virt
kvm                            # Or: vmware, xen, oracle, none

# DMI approach
$ sudo dmidecode -s system-product-name
Virtual Machine               # VirtualBox
VMware Virtual Platform        # VMware

# Indicators of VM:
$ lspci | grep -i "virtual\|vmware\|qemu\|vbox"
$ dmesg | grep -i "virtual\|vmware\|hypervisor"
```

### Full Hardware Inventory Script
```bash
#!/bin/bash
# hw_inventory.sh — Complete hardware report
echo "=== HARDWARE INVENTORY — $(date) ==="
echo ""
echo "=== SYSTEM ==="
sudo dmidecode -t system | grep -E "Manufacturer|Product|Serial" 2>/dev/null
echo ""
echo "=== CPU ==="
lscpu | grep -E "Model name|Socket|Core|Thread|CPU\(s\)|MHz|Virtualization"
echo ""
echo "=== MEMORY ==="
free -h | head -2
echo ""
sudo dmidecode -t memory | grep -E "Size|Type:|Speed|Manufacturer" | grep -v "No Module" 2>/dev/null
echo ""
echo "=== STORAGE ==="
lsblk -d -o NAME,SIZE,MODEL,TRAN,ROTA
echo ""
echo "=== NETWORK ==="
sudo lshw -class network -short 2>/dev/null
echo ""
echo "=== GPU ==="
lspci | grep -i "vga\|3d\|display"
echo ""
echo "=== CPU VULNERABILITIES ==="
for v in /sys/devices/system/cpu/vulnerabilities/*; do
    echo "  $(basename $v): $(cat $v)"
done
```

### HTML hardware report
```bash
$ sudo lshw -html > /tmp/hw_report.html
# Open in browser for professional report!
```

---

## 💡 Real World Pro Tips

### Tip 1: Quick CPU count
```bash
$ nproc                    # Number of available CPUs
16
$ nproc --all              # Total CPUs (including offline)
```

### Tip 2: Check if you're in a VM
```bash
$ systemd-detect-virt
# "none" = bare metal, anything else = virtualized
```

### Tip 3: Find your RAM speed and type
```bash
$ sudo dmidecode -t memory | grep -E "Type:|Speed:" | head -4
Type: DDR4
Speed: 3200 MT/s
```

### Tip 4: Generate professional hardware report
```bash
$ sudo lshw -html > hardware_report.html
```

---

## ✅ Pros & Cons

### `lscpu`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| No root needed | CPU only (no other hardware) |
| JSON output | No RAM/disk info |
| Shows vulnerabilities | — |
| Fast and simple | — |

### `lshw`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Full hardware inventory | Requires root for full info |
| HTML/XML/JSON export | Slow on large systems |
| Class filtering | May need installation |
| Bus information | Verbose default output |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| CPU info | `lscpu` | Architecture, cores, cache |
| Full hardware | `sudo lshw -short` | Complete inventory |
| RAM details | `sudo dmidecode -t memory` | Modules, speed |
| GPU check | `lspci \| grep VGA` | Graphics card |
| VM detection | `systemd-detect-virt` | CTF/forensics |
| CPU vulns | `grep -r . vulnerabilities/` | Security audit |
| Network HW | `sudo lshw -class network` | NIC details |
| USB devices | `lsusb` | Connected peripherals |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Running lshw without sudo | `sudo lshw` for full info |
| Confusing CPUs with cores | CPUs = sockets × cores × threads |
| Not checking CPU vulns | Check `/sys/.../vulnerabilities/` |
| Ignoring VM detection | `systemd-detect-virt` in CTF |
| Using lscpu for RAM info | Use `free -h` or `dmidecode` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Run `lscpu` and identify cores, threads, and architecture
2. Get compact hardware list with `lshw -short`
3. List PCI and USB devices

### 🟡 Intermediate
4. Check CPU vulnerabilities (Spectre, Meltdown)
5. Find RAM type and speed with `dmidecode`
6. Detect if you're in a VM

### 🔴 Advanced
7. Write a full hardware inventory script
8. Generate an HTML hardware report with `lshw`
9. Parse `lscpu -J` with `jq` for automation

---

## 🧠 Cheat Sheet

```
CPU:
  lscpu                        → CPU overview
  lscpu -e                     → Extended table
  lscpu -C                     → Cache info
  nproc                        → CPU count
  cat /proc/cpuinfo            → Raw CPU data

FULL HARDWARE:
  sudo lshw -short             → Compact summary
  sudo lshw -class cpu         → CPU only
  sudo lshw -class memory      → RAM only
  sudo lshw -class disk        → Disks only
  sudo lshw -class network     → NICs only
  sudo lshw -html > report.html → HTML report

BIOS/FIRMWARE:
  sudo dmidecode -t system     → System info
  sudo dmidecode -t bios       → BIOS version
  sudo dmidecode -t memory     → RAM modules

PCI/USB:
  lspci                        → PCI devices
  lspci | grep VGA             → GPU
  lsusb                        → USB devices

SECURITY:
  grep -r . /sys/.../vulnerabilities/  → CPU vulns
  systemd-detect-virt                  → VM detection
  lscpu | grep Hypervisor             → Hypervisor check
```

---

> **Previous**: [`free` ←](./54_free.md) | **Next**: [`tar` →](../09_compression/56_tar.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
