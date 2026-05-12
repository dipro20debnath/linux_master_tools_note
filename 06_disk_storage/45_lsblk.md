# 🛠️ `lsblk` — List Block Devices | Linux Master Note

> **Instantly see all disks, partitions, and their mount points in a beautiful tree view. `lsblk` is your X-ray into storage hardware.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--block-devices)
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

### What is `lsblk`?
`lsblk` (**L**i**s**t **Bl**oc**k** devices) displays information about all block devices (disks, partitions, USB drives, loop devices) in a tree format. It reads from the `/sys` filesystem.

### Why it matters:
- **Quick disk overview** — See all disks and partitions instantly
- **Mount point mapping** — Which partition is mounted where
- **Before partitioning** — Identify the right disk
- **Troubleshooting** — Verify disk detection and layout
- **Forensics** — Enumerate attached storage

---

## 📖 Theory — Block Devices

### What is a block device?
A block device provides **buffered access to hardware** in fixed-size blocks (typically 512 bytes or 4KB). Disks, SSDs, USB drives, and CD/DVDs are all block devices.

### Device types:
| Type | Code | Example |
|------|------|---------|
| **Disk** | `disk` | `/dev/sda` |
| **Partition** | `part` | `/dev/sda1` |
| **Loop** | `loop` | `/dev/loop0` (ISO mounts) |
| **LVM** | `lvm` | `/dev/mapper/vg-lv` |
| **RAID** | `raid` | `/dev/md0` |
| **ROM** | `rom` | `/dev/sr0` (CD/DVD) |

### Major/Minor numbers:
```
sda  → Major: 8, Minor: 0   (disk)
sda1 → Major: 8, Minor: 1   (partition 1)
sda2 → Major: 8, Minor: 2   (partition 2)
sdb  → Major: 8, Minor: 16  (second disk)
```

---

## 🧰 Syntax & Options

```bash
lsblk [OPTIONS] [DEVICE...]
```

| Flag | Description |
|------|-------------|
| `-a` | Show all devices (including empty) |
| `-f` | Show **filesystem** info (type, UUID, label, usage) |
| `-m` | Show **permissions** (owner, group, mode) |
| `-o COLUMNS` | Specify output columns |
| `-p` | Show **full device paths** |
| `-l` | List format (not tree) |
| `-t` | Show **topology** info (alignment, sizes) |
| `-d` | Show **disks only** (no partitions) |
| `-n` | No header |
| `-r` | Raw output |
| `-J` | **JSON** output |
| `-P` | Key=Value pairs output |
| `-S` | Show SCSI devices |
| `-b` | Size in bytes |
| `-e MAJOR` | Exclude devices by major number |
| `-I MAJOR` | Include only devices by major number |
| `-x COLUMN` | Sort by column |

### Available columns (`lsblk -h` for full list):
| Column | Description |
|--------|-------------|
| `NAME` | Device name |
| `SIZE` | Size of device |
| `TYPE` | Type (disk, part, lvm, loop) |
| `MOUNTPOINT` / `MOUNTPOINTS` | Where mounted |
| `FSTYPE` | Filesystem type |
| `UUID` | Filesystem UUID |
| `LABEL` | Filesystem label |
| `MODEL` | Disk model (brand) |
| `SERIAL` | Serial number |
| `VENDOR` | Manufacturer |
| `ROTA` | 1=HDD (rotational), 0=SSD |
| `RO` | Read-only |
| `RM` | Removable |
| `TRAN` | Transport (sata, usb, nvme) |
| `HOTPLUG` | Hotpluggable |
| `MAJ:MIN` | Major:Minor number |
| `OWNER` | Owner |
| `GROUP` | Group |
| `MODE` | Permissions |

---

## 🟢 Basic Usage

```bash
# Default tree view (THE command you'll use)
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   100G  0 disk
├─sda1   8:1    0   512M  0 part /boot/efi
├─sda2   8:2    0    50G  0 part /
└─sda3   8:3    0  49.5G  0 part /home
sdb      8:16   0   500G  0 disk
└─sdb1   8:17   0   500G  0 part /data
sr0     11:0    1  1024M  0 rom

# With filesystem info (type, UUID, label)
$ lsblk -f
NAME   FSTYPE FSVER LABEL UUID                                 MOUNTPOINTS
sda
├─sda1 vfat   FAT32       ABCD-1234                            /boot/efi
├─sda2 ext4   1.0         a1b2c3d4-e5f6-7890-abcd-ef1234567890 /
└─sda3 ext4   1.0         f1e2d3c4-b5a6-0987-6543-210fedcba987 /home

# Disks only (no partitions)
$ lsblk -d
NAME MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda    8:0    0  100G  0 disk
sdb    8:16   0  500G  0 disk

# Full device paths
$ lsblk -p
/dev/sda
├─/dev/sda1
├─/dev/sda2
└─/dev/sda3
```

---

## 🟡 Intermediate Usage

### Custom columns
```bash
# Show disk model and transport
$ lsblk -o NAME,SIZE,TYPE,MODEL,TRAN,ROTA
NAME   SIZE TYPE MODEL            TRAN ROTA
sda    100G disk Samsung SSD 860  sata    0  ← SSD (ROTA=0)
sdb    500G disk WDC WD5000       sata    1  ← HDD (ROTA=1)
nvme0  1TB  disk Samsung 970 EVO  nvme    0

# Show UUID for fstab entries
$ lsblk -o NAME,SIZE,FSTYPE,UUID,MOUNTPOINTS

# Show owner and permissions
$ lsblk -m
NAME   SIZE OWNER GROUP MODE
sda    100G root  disk  brw-rw----
├─sda1 512M root  disk  brw-rw----
├─sda2  50G root  disk  brw-rw----
└─sda3  50G root  disk  brw-rw----

# Identify SSDs vs HDDs
$ lsblk -d -o NAME,SIZE,ROTA,MODEL
NAME  SIZE ROTA MODEL
sda   100G    0 Samsung SSD      ← 0 = SSD
sdb   500G    1 WDC WD5000       ← 1 = HDD (rotational)
```

### JSON output (scripting)
```bash
$ lsblk -J
{
   "blockdevices": [
      {"name":"sda", "size":"100G", "type":"disk",
       "children": [
          {"name":"sda1", "size":"512M", "type":"part", "mountpoint":"/boot/efi"},
          {"name":"sda2", "size":"50G", "type":"part", "mountpoint":"/"}
       ]
      }
   ]
}

# Parse with jq
$ lsblk -J | jq '.blockdevices[] | .name, .size'
```

### Exclude loop devices (cleaner output)
```bash
# Loop devices from snaps clutter output
$ lsblk -e 7         # Exclude major number 7 (loop)
$ lsblk -e 7,11      # Exclude loop and CD-ROM
```

### Specific device
```bash
$ lsblk /dev/sda
$ lsblk -f /dev/sdb
```

---

## 🔴 Advanced Usage

### Storage Inventory Script
```bash
#!/bin/bash
# storage_inventory.sh — Complete storage report
echo "=== STORAGE INVENTORY — $(date) ==="
echo ""
echo "=== PHYSICAL DISKS ==="
lsblk -d -o NAME,SIZE,MODEL,SERIAL,TRAN,ROTA,TYPE 2>/dev/null
echo ""
echo "=== PARTITION LAYOUT ==="
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS -e 7
echo ""
echo "=== DISK USAGE ==="
df -hT -x tmpfs -x devtmpfs -x squashfs
echo ""
echo "=== FILESYSTEM UUIDs ==="
lsblk -o NAME,UUID,LABEL,FSTYPE -e 7
```

### Detect USB drives
```bash
# Show only removable devices
$ lsblk -o NAME,SIZE,RM,TYPE,TRAN,MOUNTPOINTS | awk '$3==1 || NR==1'

# Show USB devices
$ lsblk -o NAME,SIZE,TRAN,MODEL | grep usb

# Monitor USB insertion
$ watch -n 2 'lsblk -o NAME,SIZE,TRAN,MODEL | grep usb'
```

### LVM visualization
```bash
$ lsblk
sda                  100G disk
├─sda1               512M part  /boot
└─sda2                99G part
  ├─vg--system-root   50G lvm   /
  ├─vg--system-swap    8G lvm   [SWAP]
  └─vg--system-home   41G lvm   /home
```

### Forensics — Device Enumeration 🔒
```bash
# Full device inventory for evidence
$ lsblk -o NAME,SIZE,TYPE,FSTYPE,UUID,SERIAL,MODEL,VENDOR -a

# Check for hidden/suspicious partitions
$ lsblk -a       # Shows empty devices too

# Identify unknown devices
$ lsblk -S       # SCSI device info
$ lsblk -t       # Topology info

# Compare before/after (detect new devices)
$ lsblk -o NAME,SIZE,TYPE > before.txt
# ... time passes ...
$ lsblk -o NAME,SIZE,TYPE > after.txt
$ diff before.txt after.txt
```

### SCSI device info
```bash
$ lsblk -S
NAME HCTL       TYPE VENDOR   MODEL            REV  TRAN
sda  0:0:0:0    disk ATA      Samsung SSD 860  2B6Q sata
sdb  1:0:0:0    disk ATA      WDC WD5000AAKX   1H18 sata
sr0  3:0:0:0    rom  HL-DT-ST DVDRAM GH24NSC0  LY00 sata
```

---

## 🔗 Piping & Combining

```bash
# Get list of mounted partitions
$ lsblk -ln -o NAME,MOUNTPOINTS | awk '$2!=""'

# Get all UUIDs for fstab
$ lsblk -o NAME,UUID,FSTYPE,MOUNTPOINTS | grep -v "^$"

# Count partitions per disk
$ lsblk -l -o TYPE | grep part | wc -l

# Find unmounted partitions
$ lsblk -o NAME,FSTYPE,MOUNTPOINTS | awk '$2!="" && $3==""'

# Total disk space
$ lsblk -dbn -o SIZE | awk '{sum+=$1} END {printf "Total: %.1f GB\n", sum/1073741824}'

# Use with blkid for complete info
$ sudo blkid
$ sudo lsblk -f
```

---

## 💡 Real World Pro Tips

### Tip 1: Use `-e 7` to hide snap loop devices
```bash
$ lsblk -e 7    # Much cleaner on Ubuntu!
```

### Tip 2: Identify the right disk before partitioning!
```bash
# ALWAYS run lsblk before fdisk/parted
$ lsblk -o NAME,SIZE,MODEL,TRAN
# Verify you're targeting the correct disk!
```

### Tip 3: Quick fstab helper
```bash
# Generate fstab-ready UUIDs
$ lsblk -o UUID,FSTYPE,MOUNTPOINTS -n | grep -v "^$"
```

### Tip 4: Combine with related tools
```bash
$ lsblk -f             # Filesystem info
$ sudo blkid            # UUID, TYPE, LABEL
$ sudo fdisk -l         # Partition details
$ df -hT                # Usage info
$ findmnt               # Mount tree
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Beautiful tree output | Doesn't show disk usage % |
| Fast (reads /sys) | Some columns need root |
| JSON output for scripts | Loop devices clutter output |
| Custom column selection | Can't modify anything |
| Shows LVM/RAID hierarchy | Model info may be empty (VMs) |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Quick disk overview | `lsblk` | Tree view |
| Before partitioning | `lsblk -o NAME,SIZE,MODEL` | Identify correct disk |
| Filesystem UUIDs | `lsblk -f` | For fstab |
| SSD vs HDD check | `lsblk -d -o NAME,ROTA` | ROTA column |
| USB detection | `lsblk -o NAME,TRAN` | Transport type |
| Scripting | `lsblk -J` | JSON output |
| Forensics | `lsblk -a -o NAME,SIZE,TYPE,SERIAL` | Full inventory |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Cluttered output (snap loops) | Use `-e 7` to exclude |
| Confusing lsblk with df | lsblk=devices, df=usage |
| Missing model info in VMs | Normal — VMs don't have physical models |
| Not checking before partitioning | Always `lsblk` first! |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Run `lsblk` and understand the tree view
2. Show filesystem types with `lsblk -f`
3. Identify which partitions are mounted

### 🟡 Intermediate
4. Create a custom column view showing model and transport
5. Identify SSDs vs HDDs using the ROTA column
6. Use JSON output and parse with `jq`

### 🔴 Advanced
7. Write a storage inventory script
8. Detect and monitor USB device insertion
9. Find unmounted partitions for forensic analysis

---

## 🧠 Cheat Sheet

```
lsblk                        → Tree view (default)
lsblk -f                     → Filesystem info (UUID, type)
lsblk -d                     → Disks only (no partitions)
lsblk -e 7                   → Exclude loop devices
lsblk -p                     → Full device paths
lsblk -m                     → Permissions/ownership
lsblk -J                     → JSON output
lsblk -S                     → SCSI device info

CUSTOM COLUMNS:
  lsblk -o NAME,SIZE,TYPE,MODEL,TRAN,ROTA
  lsblk -o NAME,UUID,FSTYPE,MOUNTPOINTS

IDENTIFY:
  ROTA=0 → SSD    ROTA=1 → HDD
  RM=1 → Removable    RM=0 → Fixed
  TRAN: sata, usb, nvme, virtio

RELATED TOOLS:
  sudo blkid       → UUIDs and types
  sudo fdisk -l    → Partition details
  df -hT           → Disk usage
  findmnt          → Mount tree
```

---

> **Previous**: [`fdisk/parted` ←](./44_fdisk_parted.md) | **Next**: [`apt/yum` →](../07_package_management/46_apt_yum.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
