# 🛠️ `snap` & `flatpak` — Universal Package Formats | Linux Master Note

> **Cross-distro packaging. Install the same app on Ubuntu, Fedora, Arch — no dependency hell, sandboxed, and always up-to-date.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--universal-packaging)
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

### `snap` vs `flatpak` vs traditional:
| Feature | `snap` | `flatpak` | `apt`/`dnf` |
|---------|--------|-----------|-------------|
| Creator | Canonical (Ubuntu) | Red Hat / GNOME | Distro-specific |
| Store | Snap Store | Flathub | Distro repos |
| Sandboxing | ✅ AppArmor | ✅ Bubblewrap | ❌ No |
| Auto-updates | ✅ Automatic | ✅ Automatic | Manual |
| Cross-distro | ✅ Yes | ✅ Yes | ❌ No |
| Server/CLI apps | ✅ Yes | ⚠️ Mostly desktop | ✅ Yes |
| Bundle size | Large | Large | Small |
| Startup time | Slower (first launch) | Moderate | Fast |
| Default on | Ubuntu | Fedora/others | All |

---

## 📖 Theory — Universal Packaging

### How snaps work:
```
Snap Package (.snap)
├── Application binary
├── All dependencies (bundled!)
├── Runtime libraries
└── Metadata (snap.yaml)
     │
     ▼ Mounted as squashfs
/snap/appname/current/
     │
     ▼ Sandboxed via AppArmor + seccomp
     Restricted filesystem/network access
```

### How flatpaks work:
```
Flatpak App
├── Application
├── Runtime (shared base: GNOME, KDE)
│   └── Shared between apps (saves space!)
└── Permissions manifest
     │
     ▼ Sandboxed via Bubblewrap + namespaces
     Portal-based access (file picker, etc.)
```

### Key difference:
- **Snap** bundles EVERYTHING → larger but more independent
- **Flatpak** shares runtimes → smaller per app but needs runtimes installed

---

## 🧰 Syntax & Options

### `snap`:
| Command | Description |
|---------|-------------|
| `snap install PKG` | Install snap |
| `snap remove PKG` | Remove snap |
| `snap find TERM` | Search Snap Store |
| `snap info PKG` | Package details |
| `snap list` | List installed snaps |
| `snap refresh` | Update all snaps |
| `snap refresh PKG` | Update specific snap |
| `snap revert PKG` | Rollback to previous version |
| `snap connections PKG` | Show permissions |
| `snap connect/disconnect` | Manage permissions |
| `snap disable/enable PKG` | Disable/enable snap |
| `snap changes` | Show recent changes |
| `snap services` | List snap services |

### `flatpak`:
| Command | Description |
|---------|-------------|
| `flatpak install REMOTE APP` | Install app |
| `flatpak uninstall APP` | Remove app |
| `flatpak search TERM` | Search Flathub |
| `flatpak info APP` | App details |
| `flatpak list` | List installed |
| `flatpak update` | Update all |
| `flatpak run APP` | Run app |
| `flatpak override APP` | Override permissions |
| `flatpak remotes` | List configured remotes |
| `flatpak remote-add` | Add repository |
| `flatpak repair` | Fix installation issues |

---

## 🟢 Basic Usage

### Snap
```bash
# Install
$ sudo snap install vlc
$ sudo snap install code --classic        # Classic = full system access
$ sudo snap install firefox

# Remove
$ sudo snap remove vlc

# Search
$ snap find "video player"

# List installed
$ snap list
Name     Version  Rev  Tracking  Publisher  Notes
vlc      3.0.18   123  stable    videolan   -
code     1.85.0   456  stable    vscode     classic

# Update all
$ sudo snap refresh

# Info about a snap
$ snap info vlc
```

### Flatpak
```bash
# Setup Flathub (one-time)
$ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Install
$ flatpak install flathub org.videolan.VLC
$ flatpak install flathub com.spotify.Client

# Remove
$ flatpak uninstall org.videolan.VLC

# Search
$ flatpak search "video player"

# List installed
$ flatpak list

# Update all
$ flatpak update

# Run
$ flatpak run org.videolan.VLC
```

---

## 🟡 Intermediate Usage

### Snap channels (release tracks)
```bash
# Install from specific channel
$ sudo snap install firefox --channel=esr/stable   # ESR version
$ sudo snap install node --channel=18/stable        # Node.js 18

# Switch channel
$ sudo snap refresh firefox --channel=latest/stable

# Available channels
$ snap info firefox | grep channels
```

### Snap confinement levels
```bash
# Check confinement
$ snap list | grep -E "classic|strict|devmode"

# Confinement types:
# strict   → Sandboxed (default, most secure)
# classic  → Full system access (like apt install)
# devmode  → Development mode (not for production)

# Install classic snap (needs --classic flag)
$ sudo snap install code --classic
```

### Snap permissions (interfaces)
```bash
# View permissions
$ snap connections vlc
Interface       Plug                  Slot
audio-playback  vlc:audio-playback    :audio-playback
camera          vlc:camera            :camera
network         vlc:network           :network
removable-media vlc:removable-media   -              # Not connected!

# Connect permission
$ sudo snap connect vlc:removable-media

# Disconnect permission
$ sudo snap disconnect vlc:camera
```

### Flatpak permissions
```bash
# View permissions
$ flatpak info --show-permissions org.videolan.VLC

# Override permissions
$ flatpak override --user --filesystem=home org.videolan.VLC
$ flatpak override --user --no-talk-name=org.freedesktop.Notifications org.videolan.VLC

# Reset overrides
$ flatpak override --user --reset org.videolan.VLC

# Manage with Flatseal (GUI tool)
$ flatpak install flathub com.github.tchx84.Flatseal
```

### Rollback snap version
```bash
# Revert to previous version
$ sudo snap revert vlc

# Check revision history
$ snap changes
```

---

## 🔴 Advanced Usage

### Snap services management
```bash
# List snap services
$ snap services
Service              Startup  Current  Notes
nextcloud.apache     enabled  active   -
nextcloud.mysql      enabled  active   -

# Stop/start service
$ sudo snap stop nextcloud
$ sudo snap start nextcloud
$ sudo snap restart nextcloud

# Check logs
$ sudo snap logs nextcloud
$ sudo snap logs nextcloud -f    # Follow
```

### Snap disk usage
```bash
# Snaps can use lots of disk! (kept revisions)
$ du -sh /var/lib/snapd/snaps/
$ snap list --all | awk '/disabled/{print $1, $3}'

# Remove old revisions
$ sudo snap set system refresh.retain=2    # Keep only 2 revisions

# Clean old revisions script
#!/bin/bash
snap list --all | awk '/disabled/{print $1, $3}' | while read name rev; do
    sudo snap remove "$name" --revision="$rev"
done
```

### Flatpak cleanup
```bash
# Remove unused runtimes
$ flatpak uninstall --unused

# Clear app data
$ rm -rf ~/.var/app/com.spotify.Client/

# Repair corrupted installation
$ flatpak repair
$ flatpak repair --user
```

### Security — Sandbox Analysis 🔒
```bash
# Check snap sandbox restrictions
$ snap run --shell vlc
# Now you're inside the snap's sandbox

# View AppArmor profile
$ cat /var/lib/snapd/apparmor/profiles/snap.vlc.*

# Check seccomp filter
$ cat /var/lib/snapd/seccomp/bpf/snap.vlc.*.src

# Flatpak sandbox inspection
$ flatpak run --command=sh org.videolan.VLC
# Inside sandbox — test what you can access
```

---

## 💡 Real World Pro Tips

### Tip 1: Reduce snap disk usage
```bash
$ sudo snap set system refresh.retain=2
# Default keeps 3 revisions — reduce to save space
```

### Tip 2: Disable snap auto-refresh during work
```bash
# Hold updates for 24 hours
$ sudo snap refresh --hold=24h

# Resume
$ sudo snap refresh --unhold
```

### Tip 3: Speed up snap startup
```bash
# First launch is slow (decompressing squashfs)
# Subsequent launches are faster
# Use warm-up in startup scripts if needed
```

### Tip 4: Flatseal for Flatpak permissions
```bash
$ flatpak install flathub com.github.tchx84.Flatseal
# GUI tool to manage all Flatpak permissions easily
```

---

## ✅ Pros & Cons

### Snap:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Cross-distro compatibility | Slow startup (squashfs) |
| Auto-updates | Large disk usage |
| Server app support | Canonical-controlled store |
| Rollback capability | Many loop devices in `mount` |
| CLI + GUI apps | Forced auto-updates |

### Flatpak:
| ✅ Pros | ❌ Cons |
|---------|---------|
| Shared runtimes (less space per app) | Mostly desktop apps |
| Community-driven (Flathub) | Runtimes need updating |
| Better desktop integration | Portal system can be confusing |
| Granular permissions | App IDs are long |
| Flatseal for permission management | No server/CLI app support |

---

## 📍 Where & When to Use

| Scenario | Recommendation | Why |
|----------|---------------|-----|
| Desktop app | Flatpak (Flathub) | Better integration |
| Server software | Snap | CLI support |
| Latest browser | Snap (Firefox on Ubuntu) | Auto-updates |
| IDE/Editor | Snap (classic) | Full system access |
| System packages | apt/dnf | Smaller, integrated |
| Isolated testing | Either | Sandboxed |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Snap eating disk space | `snap set system refresh.retain=2` + clean |
| Flatpak can't access files | Use Flatseal to add filesystem access |
| Snap classic not installing | Must use `--classic` flag |
| Too many loop devices | Normal for snaps — ignore |
| Not adding Flathub remote | `flatpak remote-add --if-not-exists flathub URL` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Install VLC via snap and flatpak, compare experience
2. List installed snaps with `snap list`
3. Search for an application on Flathub

### 🟡 Intermediate
4. Manage snap permissions with `snap connections`
5. Roll back a snap to a previous version
6. Install Flatseal and manage flatpak permissions

### 🔴 Advanced
7. Clean old snap revisions to save disk space
8. Inspect snap sandbox restrictions
9. Override flatpak permissions for filesystem access

---

## 🧠 Cheat Sheet

```
═══ SNAP ═══
snap find term                    → Search
sudo snap install pkg             → Install
sudo snap install pkg --classic   → Full access install
sudo snap remove pkg              → Remove
snap list                         → List installed
sudo snap refresh                 → Update all
sudo snap revert pkg              → Rollback
snap connections pkg              → View permissions
snap services                     → List services

═══ FLATPAK ═══
flatpak search term               → Search
flatpak install flathub APP.ID    → Install
flatpak uninstall APP.ID          → Remove
flatpak list                      → List installed
flatpak update                    → Update all
flatpak run APP.ID                → Run app
flatpak override --user --filesystem=home APP.ID → Add permission
flatpak uninstall --unused        → Clean runtimes

═══ CLEANUP ═══
sudo snap set system refresh.retain=2  → Limit revisions
flatpak uninstall --unused             → Remove unused runtimes
```

---

> **Previous**: [`dpkg/rpm` ←](./47_dpkg_rpm.md) | **Next**: [`pip` →](./49_pip.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
