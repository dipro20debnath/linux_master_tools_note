# 🛠️ `mount` & `umount` — Mount/Unmount Filesystems | Linux Master Note

> **Attach storage devices to the Linux filesystem tree. Every USB drive, network share, and disk partition must be mounted before use.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--mount-system)
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

### What is mounting?
**Mounting** attaches a filesystem (disk, partition, USB, ISO, network share) to a specific directory in the Linux directory tree. Everything in Linux is accessed through the **single unified directory tree** starting at `/`.

### Key concept:
```
Before mount:
/mnt/usb/  → empty directory

After mount:
/mnt/usb/  → contents of USB drive appear here!
```

---

## 📖 Theory — Mount System

### How mounting works:
```
Physical Device → Filesystem → Mount Point → Accessible!
/dev/sdb1       → ext4       → /mnt/data   → ls /mnt/data
```

### Key files:
| File | Purpose |
|------|---------|
| `/etc/fstab` | **Permanent** mount definitions (loaded at boot) |
| `/proc/mounts` | Currently mounted filesystems (live) |
| `/etc/mtab` | Legacy mount table |
| `/media/` | Auto-mounted removable devices (desktop) |
| `/mnt/` | Temporary mount points (by convention) |

### `/etc/fstab` format:
```
# device          mount_point  type  options       dump  fsck
/dev/sda1         /            ext4  defaults      0     1
/dev/sda2         /home        ext4  defaults      0     2
UUID=abc123...    /data        ext4  defaults      0     2
//server/share    /mnt/nas     cifs  credentials   0     0
```

### Common filesystem types:
| Type | Use Case |
|------|----------|
| `ext4` | Standard Linux filesystem |
| `xfs` | High-performance (RHEL default) |
| `btrfs` | Modern with snapshots |
| `ntfs` | Windows drives |
| `vfat` / `fat32` | USB drives, cross-platform |
| `exfat` | Large USB drives, SD cards |
| `iso9660` | CD/DVD images |
| `nfs` | Network File System |
| `cifs` / `smb` | Windows network shares |
| `tmpfs` | RAM-based filesystem |
| `squashfs` | Read-only compressed |

---

## 🧰 Syntax & Options

### `mount`:
```bash
mount [OPTIONS] DEVICE MOUNTPOINT
```

| Flag | Description |
|------|-------------|
| `-t TYPE` | Specify filesystem type |
| `-o OPTIONS` | Mount options (comma-separated) |
| `-a` | Mount all from `/etc/fstab` |
| `-r` | Mount read-only |
| `-w` | Mount read-write |
| `--bind` | Bind mount (mount directory to another location) |
| `-L LABEL` | Mount by label |
| `-U UUID` | Mount by UUID |
| `-l` | Show labels in mount list |
| `-n` | Don't write to /etc/mtab |

### Common mount options (`-o`):
| Option | Description |
|--------|-------------|
| `defaults` | rw, suid, dev, exec, auto, nouser, async |
| `ro` | Read-only |
| `rw` | Read-write |
| `noexec` | Can't execute binaries |
| `nosuid` | Ignore SUID/SGID bits |
| `nodev` | Ignore device files |
| `noatime` | Don't update access time (performance!) |
| `auto` / `noauto` | Auto-mount at boot / don't |
| `user` / `nouser` | Allow/deny non-root mounting |
| `uid=N` | Set file owner UID |
| `gid=N` | Set file group GID |
| `umask=NNN` | Set permission mask |
| `remount` | Remount with new options |

### `umount`:
```bash
umount [OPTIONS] MOUNTPOINT_or_DEVICE
```

| Flag | Description |
|------|-------------|
| `-l` | **Lazy** unmount (detach, cleanup later) |
| `-f` | **Force** unmount (NFS stale mounts) |
| `-a` | Unmount all |

---

## 🟢 Basic Usage

```bash
# View currently mounted filesystems
$ mount
$ mount | column -t     # Prettier output
$ findmnt               # Modern alternative (tree view)
$ findmnt -t ext4       # Filter by type

# Mount a USB drive
$ sudo mkdir -p /mnt/usb
$ sudo mount /dev/sdb1 /mnt/usb
$ ls /mnt/usb           # Access contents

# Mount with specific type
$ sudo mount -t ext4 /dev/sdb1 /mnt/usb
$ sudo mount -t ntfs-3g /dev/sdb1 /mnt/windows

# Unmount
$ sudo umount /mnt/usb
# OR
$ sudo umount /dev/sdb1

# Mount all from fstab
$ sudo mount -a
```

---

## 🟡 Intermediate Usage

### Mount ISO image
```bash
$ sudo mkdir -p /mnt/iso
$ sudo mount -o loop ubuntu.iso /mnt/iso
$ ls /mnt/iso       # Browse ISO contents
$ sudo umount /mnt/iso
```

### Mount with specific options
```bash
# Read-only
$ sudo mount -o ro /dev/sdb1 /mnt/evidence

# No execute (security — prevent running binaries)
$ sudo mount -o noexec,nosuid,nodev /dev/sdb1 /mnt/untrusted

# Performance (no access time updates)
$ sudo mount -o noatime /dev/sdb1 /mnt/data
```

### Remount with different options
```bash
# Remount root filesystem as read-write (recovery)
$ sudo mount -o remount,rw /

# Remount as read-only
$ sudo mount -o remount,ro /mnt/data
```

### Mount by UUID or label (recommended for fstab)
```bash
# Find UUID
$ sudo blkid
/dev/sda1: UUID="a1b2c3d4" TYPE="ext4"

# Mount by UUID
$ sudo mount UUID="a1b2c3d4" /mnt/data

# Mount by label
$ sudo mount -L "MyData" /mnt/data
```

### Persistent mount (`/etc/fstab`)
```bash
# Get UUID
$ sudo blkid /dev/sdb1

# Add to /etc/fstab
UUID=a1b2c3d4-e5f6-7890  /mnt/data  ext4  defaults,noatime  0  2

# Test without reboot
$ sudo mount -a
$ df -h /mnt/data       # Verify

# ⚠️ ALWAYS test with mount -a before rebooting!
# Bad fstab entry = system won't boot!
```

### Network mounts (NFS/CIFS)
```bash
# NFS mount
$ sudo mount -t nfs server:/shared /mnt/nfs
# fstab: server:/shared  /mnt/nfs  nfs  defaults  0  0

# Windows/Samba share (CIFS)
$ sudo mount -t cifs //server/share /mnt/smb -o username=user,password=pass
# fstab: //server/share  /mnt/smb  cifs  credentials=/root/.smbcred  0  0

# Credentials file (more secure):
$ cat /root/.smbcred
username=admin
password=secret
domain=WORKGROUP
$ chmod 600 /root/.smbcred
```

---

## 🔴 Advanced Usage

### Bind mounts
```bash
# Mount a directory to another location
$ sudo mount --bind /var/www/html /home/dipro/web_mirror

# Useful for chroot environments, containers
$ sudo mount --bind /dev /chroot/dev
$ sudo mount --bind /proc /chroot/proc
$ sudo mount --bind /sys /chroot/sys
```

### tmpfs (RAM filesystem)
```bash
# Create RAM disk (fast temporary storage)
$ sudo mount -t tmpfs -o size=2G tmpfs /mnt/ramdisk
# ⚠️ Data lost on unmount/reboot!

# fstab: tmpfs  /mnt/ramdisk  tmpfs  size=2G  0  0
```

### Forced unmount (when "device busy")
```bash
# Error: "target is busy"
$ sudo umount /mnt/usb
umount: /mnt/usb: target is busy

# Find what's using the mount
$ sudo lsof +D /mnt/usb
$ sudo fuser -vm /mnt/usb

# Kill processes using it
$ sudo fuser -km /mnt/usb

# Lazy unmount (detaches, cleans up when free)
$ sudo umount -l /mnt/usb

# Force unmount (NFS stale mounts)
$ sudo umount -f /mnt/nfs
```

### Forensics — Read-Only Evidence Mount 🔒
```bash
# Mount evidence disk as read-only (CRITICAL for forensics!)
$ sudo mount -o ro,noexec,nosuid,nodev,noatime /dev/sdb1 /mnt/evidence

# Even better: use a write blocker or loopback
$ sudo losetup -r /dev/loop0 /path/to/disk.img
$ sudo mount -o ro /dev/loop0 /mnt/evidence
```

---

## 💡 Real World Pro Tips

### Tip 1: Use `findmnt` instead of `mount`
```bash
$ findmnt                # Tree view — much better!
$ findmnt -t ext4        # Filter by type
$ findmnt /home          # Find mount for path
```

### Tip 2: Always use UUID in fstab (not device names!)
```bash
# ❌ Device names can change!
/dev/sdb1  /data  ext4  defaults  0  2

# ✅ UUID is permanent
UUID=a1b2c3d4  /data  ext4  defaults  0  2
```

### Tip 3: Test fstab before reboot
```bash
$ sudo mount -a          # Test all fstab entries
$ echo $?                # 0 = success
# If errors → fix before rebooting!
```

### Tip 4: Security mount options
```bash
# For /tmp and removable media:
/dev/sdb1  /mnt/usb  ext4  nosuid,noexec,nodev  0  0
# Prevents privilege escalation via mounted media
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Unified filesystem tree | "Device busy" errors |
| Supports every filesystem | Bad fstab = unbootable system |
| Read-only for forensics | Complex NFS/CIFS options |
| Bind mounts for flexibility | Mount points must exist |
| tmpfs for RAM storage | Requires root |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| USB drive | `mount /dev/sdb1 /mnt/usb` | Access data |
| ISO image | `mount -o loop file.iso /mnt` | Browse ISO |
| Network share | `mount -t nfs/cifs` | Remote storage |
| Forensics | `mount -o ro` | Preserve evidence |
| Performance | `mount -o noatime` | Reduce I/O |
| Boot config | Edit `/etc/fstab` | Persistent |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using device names in fstab | Use UUID instead |
| Not testing fstab before reboot | `sudo mount -a` |
| "Device busy" on unmount | `lsof +D` to find users |
| Mounting evidence as rw | Always `mount -o ro` |
| Forgetting to create mount point | `mkdir -p /mnt/point` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. View all mounted filesystems with `mount` and `findmnt`
2. Mount and unmount a USB drive
3. Mount an ISO file with `-o loop`

### 🟡 Intermediate
4. Add a persistent mount to `/etc/fstab` using UUID
5. Mount a network share (NFS or CIFS)
6. Create a tmpfs RAM disk

### 🔴 Advanced
7. Set up a forensics-safe read-only mount
8. Create bind mounts for a chroot environment
9. Troubleshoot "device busy" with `lsof` and `fuser`

---

## 🧠 Cheat Sheet

```
VIEW:
  mount                      → List all mounts
  findmnt                    → Tree view (better!)
  findmnt -t ext4            → Filter by type
  sudo blkid                 → Show UUIDs

MOUNT:
  sudo mount /dev/sdb1 /mnt/usb     → Basic mount
  sudo mount -t ntfs-3g dev /mnt    → Specific type
  sudo mount -o loop file.iso /mnt  → ISO image
  sudo mount -o ro dev /mnt         → Read-only
  sudo mount -a                     → Mount all fstab
  sudo mount --bind src dst         → Bind mount

UNMOUNT:
  sudo umount /mnt/usb       → Normal unmount
  sudo umount -l /mnt/usb    → Lazy unmount
  sudo umount -f /mnt/nfs    → Force unmount
  sudo fuser -km /mnt/usb    → Kill users first

FSTAB:
  UUID=xxxx  /mount  ext4  defaults,noatime  0  2
  # Always test: sudo mount -a
```

---

> **Previous**: [`du` ←](./42_du.md) | **Next**: [`fdisk/parted` →](./44_fdisk_parted.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
