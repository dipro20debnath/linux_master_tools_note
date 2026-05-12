# 🛠️ `fdisk` & `parted` — Partition Table Manipulator | Linux Master Note

> **Create, delete, and manage disk partitions. `fdisk` is the classic MBR tool, `parted` handles both MBR and GPT — the foundation of disk management.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--partition-tables)
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

### `fdisk` vs `parted`:
| Feature | `fdisk` | `parted` |
|---------|---------|---------|
| MBR support | ✅ Yes | ✅ Yes |
| GPT support | ✅ (modern versions) | ✅ Yes |
| Interface | Interactive menu | Interactive + CLI |
| Disk > 2TB | ❌ No (MBR limit) | ✅ Yes (GPT) |
| Write changes | On exit (`w`) | **Immediately!** |
| Resize partitions | ❌ No | ✅ Yes |

> ⚠️ **DANGER ZONE**: These tools modify partition tables. Wrong commands = **DATA LOSS!**

---

## 📖 Theory — Partition Tables

### MBR vs GPT:
| Feature | MBR | GPT |
|---------|-----|-----|
| Max disk size | 2 TB | 9.4 ZB (zettabytes) |
| Max partitions | 4 primary (or 3 + extended) | 128 |
| Boot mode | BIOS (Legacy) | UEFI |
| Redundancy | No backup | Backup header at end of disk |
| OS support | All | Modern OS only |

### Partition types:
| Type | Description |
|------|-------------|
| **Primary** | Bootable, max 4 per disk (MBR) |
| **Extended** | Container for logical partitions (MBR) |
| **Logical** | Inside extended partition (MBR) |
| **GPT partition** | No primary/extended distinction |

### Linux device naming:
```
/dev/sda     → First SATA/SCSI disk
/dev/sda1    → First partition on sda
/dev/sda2    → Second partition on sda
/dev/sdb     → Second disk
/dev/nvme0n1 → First NVMe SSD
/dev/nvme0n1p1 → First partition on NVMe
/dev/vda     → Virtual disk (KVM)
```

### Common filesystem IDs (MBR):
| ID | Type |
|----|------|
| `83` | Linux |
| `82` | Linux swap |
| `8e` | Linux LVM |
| `fd` | Linux RAID |
| `7` | NTFS/Windows |
| `c` | FAT32 (LBA) |
| `ef` | EFI System |

---

## 🧰 Syntax & Options

### `fdisk`:
```bash
fdisk [OPTIONS] DEVICE
```

| Flag | Description |
|------|-------------|
| `-l` | **List** all partition tables |
| `-u` | Display in sectors |
| `-c` | Disable DOS compatibility |

### `fdisk` interactive commands:
| Key | Action |
|-----|--------|
| `m` | Help menu |
| `p` | Print partition table |
| `n` | **New** partition |
| `d` | **Delete** partition |
| `t` | Change partition **type** |
| `l` | List known partition types |
| `a` | Toggle bootable flag |
| `w` | **Write** changes and exit |
| `q` | **Quit** without saving |
| `g` | Create new GPT table |
| `o` | Create new MBR table |

### `parted`:
```bash
parted [OPTIONS] [DEVICE] [COMMAND]
```

| Command | Description |
|---------|-------------|
| `print` | Show partition table |
| `mklabel TYPE` | Create partition table (gpt/msdos) |
| `mkpart TYPE START END` | Create partition |
| `rm NUMBER` | Delete partition |
| `resizepart N END` | Resize partition |
| `name N NAME` | Name a partition (GPT) |
| `set N FLAG on/off` | Set flag (boot, lvm, raid) |
| `unit s/MB/GB/%` | Set display units |

---

## 🟢 Basic Usage

### View partition table
```bash
# List all disks and partitions
$ sudo fdisk -l
Disk /dev/sda: 100 GiB
Device     Boot   Start       End   Sectors  Size Id Type
/dev/sda1  *       2048   1026047   1024000  500M 83 Linux
/dev/sda2       1026048 209715199 208689152 99.5G 8e Linux LVM

# Specific disk
$ sudo fdisk -l /dev/sda

# Using parted
$ sudo parted /dev/sda print
Number  Start   End    Size   Type     File system  Flags
 1      1049kB  526MB  525MB  primary  ext4         boot
 2      526MB   107GB  107GB  primary               lvm
```

---

## 🟡 Intermediate Usage

### Create partition with fdisk
```bash
$ sudo fdisk /dev/sdb

# Inside fdisk:
Command: n            # New partition
Partition type: p     # Primary
Partition number: 1
First sector: [Enter] # Default
Last sector: +20G     # 20GB partition

Command: t            # Change type
Hex code: 83          # Linux

Command: p            # Print to verify
Command: w            # Write and exit

# Create filesystem
$ sudo mkfs.ext4 /dev/sdb1

# Mount it
$ sudo mkdir -p /mnt/data
$ sudo mount /dev/sdb1 /mnt/data
```

### Create partition with parted
```bash
# Non-interactive (one command)
$ sudo parted /dev/sdb mklabel gpt
$ sudo parted /dev/sdb mkpart primary ext4 0% 50%
$ sudo parted /dev/sdb mkpart primary ext4 50% 100%

# Interactive
$ sudo parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart primary ext4 1MiB 20GiB
(parted) mkpart primary ext4 20GiB 100%
(parted) print
(parted) quit
```

### Create different filesystems
```bash
$ sudo mkfs.ext4 /dev/sdb1       # Linux standard
$ sudo mkfs.xfs /dev/sdb2        # High performance
$ sudo mkfs.btrfs /dev/sdb3      # Modern with snapshots
$ sudo mkfs.vfat -F32 /dev/sdb4  # FAT32 (USB drives)
$ sudo mkfs.ntfs /dev/sdb5       # Windows compatible
$ sudo mkswap /dev/sdb6          # Swap partition
```

### Delete partition
```bash
# fdisk
$ sudo fdisk /dev/sdb
Command: d
Partition number: 1
Command: w

# parted
$ sudo parted /dev/sdb rm 1
```

---

## 🔴 Advanced Usage

### Create GPT partition table (UEFI boot)
```bash
$ sudo fdisk /dev/sda
Command: g              # Create GPT
Command: n              # EFI System partition
Partition number: 1
Size: +512M
Command: t
Type: 1                 # EFI System
Command: n              # Root partition
Partition number: 2
Size: +50G
Command: n              # Home partition
Partition number: 3
Size: [remaining]
Command: w

# Format
$ sudo mkfs.fat -F32 /dev/sda1    # EFI must be FAT32
$ sudo mkfs.ext4 /dev/sda2        # Root
$ sudo mkfs.ext4 /dev/sda3        # Home
```

### LVM (Logical Volume Manager)
```bash
# Create physical volumes
$ sudo pvcreate /dev/sdb1 /dev/sdc1

# Create volume group
$ sudo vgcreate datavg /dev/sdb1 /dev/sdc1

# Create logical volumes
$ sudo lvcreate -L 50G -n data_lv datavg
$ sudo lvcreate -l 100%FREE -n backup_lv datavg

# Format and mount
$ sudo mkfs.ext4 /dev/datavg/data_lv
$ sudo mount /dev/datavg/data_lv /mnt/data

# Extend later (LVM's superpower!)
$ sudo lvextend -L +20G /dev/datavg/data_lv
$ sudo resize2fs /dev/datavg/data_lv
```

### Swap management
```bash
# Create swap partition
$ sudo mkswap /dev/sda3
$ sudo swapon /dev/sda3

# Create swap file (alternative)
$ sudo fallocate -l 4G /swapfile
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile

# fstab entry:
/swapfile  none  swap  sw  0  0

# Check swap
$ swapon --show
$ free -h
```

### Forensics — Disk Analysis 🔒
```bash
# View disk without modifying
$ sudo fdisk -l /dev/sdb
$ sudo parted /dev/sdb print

# Check for hidden partitions (gaps between partitions)
$ sudo fdisk -l /dev/sdb | grep -E "Start|/dev/"

# Recover deleted partition
$ sudo testdisk /dev/sdb      # Interactive recovery tool

# Wipe disk securely
$ sudo dd if=/dev/zero of=/dev/sdb bs=1M status=progress
$ sudo shred -vfz -n 3 /dev/sdb   # DOD-level wipe
```

---

## 💡 Real World Pro Tips

### Tip 1: Always backup partition table!
```bash
# Backup MBR
$ sudo dd if=/dev/sda of=sda_mbr.bak bs=512 count=1

# Backup GPT
$ sudo sgdisk --backup=sda_gpt.bak /dev/sda

# Restore
$ sudo dd if=sda_mbr.bak of=/dev/sda bs=512 count=1
$ sudo sgdisk --load-backup=sda_gpt.bak /dev/sda
```

### Tip 2: Use `parted` for GPT, `fdisk` for MBR
```bash
# Check what table type exists
$ sudo fdisk -l /dev/sda | grep "Disklabel"
Disklabel type: gpt    # or: dos (MBR)
```

### Tip 3: Align partitions for SSD performance
```bash
# parted auto-aligns, but verify:
$ sudo parted /dev/sda align-check optimal 1
1 aligned
```

### Tip 4: `parted` writes immediately — BE CAREFUL!
```bash
# fdisk: changes applied ONLY when you press 'w'
# parted: changes applied IMMEDIATELY!
# Use parted's -s flag for scripts, always double-check
```

---

## ✅ Pros & Cons

### `fdisk`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Changes only on 'w' (safe) | Originally MBR only |
| Simple interactive menu | No resize capability |
| Pre-installed everywhere | No scripting mode |

### `parted`:
| ✅ Pros | ❌ Cons |
|---------|---------|
| GPT + MBR support | Changes are IMMEDIATE |
| Can resize partitions | More complex syntax |
| Scriptable | Can be dangerous |
| Handles large disks | — |

---

## 📍 Where & When to Use

| Scenario | Tool | Why |
|----------|------|-----|
| View partitions | `fdisk -l` or `parted print` | Quick overview |
| Simple MBR partitioning | `fdisk` | Safe (w to save) |
| GPT partitioning | `parted` or `gdisk` | Full support |
| Disk > 2TB | `parted` (GPT) | MBR can't handle |
| Scripted partitioning | `parted` or `sfdisk` | Non-interactive |
| LVM setup | `pvcreate/vgcreate/lvcreate` | Flexible volumes |
| Forensics | `fdisk -l` (read-only) | Non-destructive |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Partitioning the wrong disk! | Double-check with `lsblk` first |
| Using MBR for disks > 2TB | Use GPT |
| Not creating filesystem after partition | `mkfs.ext4 /dev/sdX1` |
| Forgetting to update fstab | Add UUID entry |
| parted changes are instant | Triple-check before commands |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. List all partitions with `fdisk -l`
2. View partition table with `parted print`
3. Identify disk types (MBR vs GPT)

### 🟡 Intermediate
4. Create a partition on a spare disk with `fdisk`
5. Format with ext4 and mount it
6. Add persistent mount to `/etc/fstab`

### 🔴 Advanced
7. Create GPT layout with EFI + root + home + swap
8. Set up LVM with multiple disks
9. Backup and restore a partition table

---

## 🧠 Cheat Sheet

```
VIEW:
  sudo fdisk -l              → List all partitions
  sudo parted /dev/sda print → Partition table
  sudo blkid                 → UUIDs and types

FDISK (interactive):
  n → New    d → Delete    t → Type
  p → Print  w → Write     q → Quit
  g → New GPT   o → New MBR

PARTED:
  sudo parted /dev/sdb mklabel gpt
  sudo parted /dev/sdb mkpart primary ext4 1MiB 20GiB
  sudo parted /dev/sdb rm 1

FILESYSTEM:
  sudo mkfs.ext4 /dev/sdb1   → Create ext4
  sudo mkfs.xfs /dev/sdb1    → Create xfs
  sudo mkswap /dev/sdb2      → Create swap

LVM:
  pvcreate → vgcreate → lvcreate → mkfs → mount

BACKUP:
  sudo dd if=/dev/sda of=mbr.bak bs=512 count=1
  sudo sgdisk --backup=gpt.bak /dev/sda
```

---

> **Previous**: [`mount/umount` ←](./43_mount_umount.md) | **Next**: [`lsblk` →](./45_lsblk.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
