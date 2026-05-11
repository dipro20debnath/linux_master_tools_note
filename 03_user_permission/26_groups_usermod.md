# 🛠️ `groups` & `usermod` — Manage User Groups | Linux Master Note

> **Groups are the backbone of Linux access control. Master group management and you control who can access what across the entire system.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--linux-group-system)
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

### What are Groups?
Groups organize users into logical collections for **shared access control**. Every file has an owner AND a group — group members get the group's permissions.

### Key Commands:
| Command | Purpose |
|---------|---------|
| `groups` | Show user's group memberships |
| `usermod` | Modify user account (including groups) |
| `groupadd` | Create new group |
| `groupdel` | Delete group |
| `groupmod` | Modify group |
| `gpasswd` | Administer groups |
| `newgrp` | Switch active group |
| `id` | Show UID, GID, all groups |
| `getent group` | Query group database |

### Why it matters:
- **Team collaboration** — Shared project directories
- **Service access** — `docker`, `sudo`, `www-data` groups
- **Security segmentation** — Principle of least privilege
- **Compliance** — Role-based access control (RBAC)

---

## 📖 Theory — Linux Group System

### Two types of groups:

| Type | Description | Example |
|------|-------------|---------|
| **Primary group** | User's default group (GID in `/etc/passwd`) | Files created get this group |
| **Supplementary groups** | Additional groups user belongs to | `sudo`, `docker`, `developers` |

### Where group data lives:

| File | Purpose | Format |
|------|---------|--------|
| `/etc/group` | Group definitions | `groupname:x:GID:member1,member2` |
| `/etc/gshadow` | Group passwords | `groupname:password:admins:members` |
| `/etc/passwd` | Primary GID per user | `...1000:1000:...` (4th field) |

### `/etc/group` format:
```
developers:x:1001:dipro,alice,bob
    │       │  │     │
    │       │  │     └─ Supplementary members (comma-separated)
    │       │  └─ GID (Group ID)
    │       └─ Password placeholder (in /etc/gshadow)
    └─ Group name
```

### GID Ranges:
| Range | Type | Examples |
|-------|------|---------|
| `0` | Root group | `root` |
| `1-999` | System groups | `www-data`(33), `sudo`(27), `docker`(999) |
| `1000+` | User groups | `dipro`(1000), `developers`(1001) |

### How groups affect file access:
```bash
$ ls -la project/
-rw-rw-r-- 1 dipro developers 4096 app.py
#                  ├────────┤
# Group: developers — all members of 'developers' get rw access
```

---

## 🧰 Syntax & Options

### `groups` syntax:
```bash
groups [USERNAME]     # Show groups for user (default: current user)
```

### `usermod` syntax:
```bash
usermod [OPTIONS] USERNAME
```

| Flag | Description |
|------|-------------|
| `-aG GROUP` | **Append** to supplementary group (CRITICAL: always use `-a`!) |
| `-G grp1,grp2` | Set supplementary groups (⚠️ REPLACES all groups!) |
| `-g GROUP` | Change primary group |
| `-l NEWNAME` | Change username |
| `-d /path` | Change home directory |
| `-m` | Move home directory contents (use with `-d`) |
| `-s /bin/shell` | Change login shell |
| `-L` | Lock account |
| `-U` | Unlock account |
| `-e DATE` | Set expiration date |
| `-c "Comment"` | Change GECOS (full name) |

### `groupadd` syntax:
```bash
groupadd [OPTIONS] GROUPNAME
```

| Flag | Description |
|------|-------------|
| `-g GID` | Specify GID |
| `-r` | Create system group (GID < 1000) |
| `-f` | Force — exit successfully if group exists |

### `gpasswd` syntax:
```bash
gpasswd [OPTIONS] GROUP
```

| Flag | Description |
|------|-------------|
| `-a USER` | Add user to group |
| `-d USER` | Remove user from group |
| `-A USER` | Set group administrator |
| `-M u1,u2` | Set member list |

---

## 🟢 Basic Usage

### Check groups
```bash
# Your groups
$ groups
dipro sudo docker developers

# Another user's groups
$ groups alice
alice : alice developers webadmins

# Detailed view with UIDs/GIDs
$ id dipro
uid=1000(dipro) gid=1000(dipro) groups=1000(dipro),27(sudo),999(docker),1001(developers)
```

### Create a new group
```bash
$ sudo groupadd developers
$ sudo groupadd -g 2000 pentesting     # With specific GID
```

### Add user to group (MOST COMMON OPERATION)
```bash
# ⚠️ ALWAYS use -a (append)! Without -a, it REPLACES all groups!

# ✅ CORRECT — append to supplementary groups
$ sudo usermod -aG sudo dipro
$ sudo usermod -aG docker dipro
$ sudo usermod -aG developers dipro

# ❌ DANGEROUS — replaces ALL supplementary groups!
$ sudo usermod -G developers dipro
# Now dipro is ONLY in 'developers' — removed from sudo, docker, etc!
```

### Verify the change
```bash
$ groups dipro
dipro : dipro sudo docker developers

# Note: User must LOG OUT and LOG BACK IN for group changes to take effect!
$ newgrp developers     # OR use newgrp to activate immediately
```

---

## 🟡 Intermediate Usage

### Remove user from group
```bash
# Method 1: gpasswd (RECOMMENDED)
$ sudo gpasswd -d alice developers
Removing user alice from group developers

# Method 2: Manually set groups (replaces all — be careful!)
$ sudo usermod -G grp1,grp2 alice     # Only these groups
```

### Change primary group
```bash
$ sudo usermod -g developers dipro

# Verify
$ id dipro
uid=1000(dipro) gid=1001(developers) ...
```

### Modify existing group
```bash
# Rename group
$ sudo groupmod -n devops developers

# Change GID
$ sudo groupmod -g 2000 devops
```

### Delete group
```bash
$ sudo groupdel old_team
# ⚠️ Can't delete a user's primary group while user exists
```

### Switch active group (for file creation)
```bash
# Files you create will have this group
$ newgrp developers
$ touch project_file.txt
$ ls -la project_file.txt
-rw-r--r-- 1 dipro developers 0 May 11 project_file.txt
```

### Modify user account
```bash
# Change username
$ sudo usermod -l newname oldname

# Change home directory (and move contents)
$ sudo usermod -d /home/newpath -m username

# Change shell
$ sudo usermod -s /bin/zsh username

# Change full name
$ sudo usermod -c "Dipro Kumar Debnath" dipro
```

---

## 🔴 Advanced Usage

### Team Collaboration Setup 🏢
```bash
# 1. Create team group
$ sudo groupadd webteam

# 2. Add users
$ sudo usermod -aG webteam alice
$ sudo usermod -aG webteam bob
$ sudo usermod -aG webteam charlie

# 3. Create shared directory with SGID
$ sudo mkdir -p /projects/webapp
$ sudo chown root:webteam /projects/webapp
$ sudo chmod 2775 /projects/webapp
# SGID (2) → new files inherit 'webteam' group
# 775 → owner/group full access, others read

# 4. Verify — all files created here inherit webteam
$ cd /projects/webapp && touch test.txt
$ ls -la test.txt
-rw-rw-r-- 1 alice webteam 0 test.txt
```

### Docker Access Without sudo
```bash
# Add user to docker group
$ sudo usermod -aG docker dipro

# Log out and back in, then:
$ docker ps      # Works without sudo!
```

### Security Auditing 🔒
```bash
# List all groups
$ cat /etc/group | cut -d: -f1

# List all members of a group
$ getent group sudo
sudo:x:27:dipro,alice

# Find users in sudo/wheel group
$ getent group sudo wheel 2>/dev/null

# Find ALL groups a user belongs to
$ id -nG dipro
dipro sudo docker developers

# Find users with no supplementary groups (potential new accounts)
$ awk -F: '$4 == "" {print $1}' /etc/group

# Find groups with no members
$ awk -F: '$4 == "" {print $1":"$3}' /etc/group

# Check for suspicious group memberships
$ for user in $(awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd); do
    echo "$user: $(id -nG $user)"
done
```

### CTF/Pentesting — Group Enumeration 🎯
```bash
# Check your groups (privilege escalation clues!)
$ id
uid=1001(www-data) gid=33(www-data) groups=33(www-data),999(docker),1001(lxd)

# docker group → escape to host
$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# lxd group → privilege escalation
# adm group → can read /var/log/
# disk group → raw disk access!

# Check what files your groups can access
$ find / -group docker -readable 2>/dev/null
$ find / -group adm -readable 2>/dev/null
```

### Batch Group Management
```bash
#!/bin/bash
# add_team.sh — Add multiple users to a group
GROUP="developers"
USERS="alice bob charlie dave"

for user in $USERS; do
    sudo usermod -aG "$GROUP" "$user"
    echo "Added $user to $GROUP"
done
```

---

## 🔗 Piping & Combining

```bash
# List all users in sudo group
$ getent group sudo | cut -d: -f4 | tr ',' '\n'

# Add all users from a file to a group
$ cat users.txt | while read user; do
    sudo usermod -aG developers "$user"
done

# Find files accessible by a specific group
$ find / -group developers -type f 2>/dev/null

# Check group membership for all human users
$ awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd | while read u; do
    echo "$u: $(groups $u 2>/dev/null)"
done
```

---

## 💡 Real World Pro Tips

### Tip 1: ALWAYS use `-aG` (append), never just `-G`!
```bash
# ❌ DISASTER — removes user from ALL other groups!
$ sudo usermod -G docker dipro
# dipro loses sudo, developers, etc.!

# ✅ SAFE — appends to existing groups
$ sudo usermod -aG docker dipro
```

### Tip 2: Group changes require re-login
```bash
# After adding to group:
$ sudo usermod -aG docker dipro

# Option 1: Log out and log back in
# Option 2: Use newgrp (temporary)
$ newgrp docker

# Option 3: Use su (refresh)
$ su - dipro
```

### Tip 3: Use `getent` instead of reading files
```bash
# Works with LDAP, NIS, etc. — not just local files
$ getent group developers
developers:x:1001:dipro,alice

$ getent passwd dipro
dipro:x:1000:1000:Dipro Debnath:/home/dipro:/bin/bash
```

### Tip 4: Quick group membership check
```bash
# Am I in the docker group?
$ groups | grep -q docker && echo "YES" || echo "NO"
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Flexible group-based access | Group changes need re-login |
| Multiple groups per user | `-G` without `-a` is destructive |
| SGID for directory inheritance | Can't nest groups natively |
| Works with LDAP/AD | No built-in RBAC hierarchy |
| Simple auditing | Group membership sprawl |

---

## 📍 Where & When to Use

| Scenario | Command | Why |
|----------|---------|-----|
| Add to sudo group | `usermod -aG sudo user` | Grant admin access |
| Docker without sudo | `usermod -aG docker user` | Dev convenience |
| Team project setup | `groupadd` + SGID dir | Shared access |
| Remove from group | `gpasswd -d user group` | Revoke access |
| Security audit | `getent group sudo` | Check privileged users |
| Change username | `usermod -l new old` | Account migration |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| `usermod -G` without `-a` | ALWAYS use `-aG` to append |
| Not re-logging after group change | Logout/login or `newgrp` |
| Deleting primary group | Remove user first |
| Not auditing group membership | Regular `getent group sudo` checks |
| docker/lxd group = root equivalent | Only give to trusted users |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Check your groups with `groups` and `id`
2. Create a new group called `testteam`
3. Add yourself to the new group

### 🟡 Intermediate
4. Set up a shared directory with SGID for team access
5. Add a user to docker group and verify
6. Remove a user from a group using `gpasswd`

### 🔴 Advanced
7. Write a script to add multiple users to a group from file
8. Audit all users in the sudo group
9. Set up group-based file access with proper SGID configuration

---

## 🧠 Cheat Sheet

```
CHECK:
  groups                   → Your groups
  groups user              → User's groups
  id user                  → UID, GID, all groups
  getent group grpname     → Group members

CREATE/DELETE:
  sudo groupadd mygroup    → Create group
  sudo groupdel mygroup    → Delete group
  sudo groupmod -n new old → Rename group

ADD/REMOVE USERS:
  sudo usermod -aG grp user    → Add to group (ALWAYS -a!)
  sudo gpasswd -d user grp    → Remove from group

MODIFY USER:
  sudo usermod -l new old      → Rename user
  sudo usermod -d /path -m u   → Move home
  sudo usermod -s /bin/zsh u   → Change shell
  sudo usermod -L user         → Lock account
  sudo usermod -U user         → Unlock account

SHARED DIR SETUP:
  sudo groupadd team
  sudo usermod -aG team user
  sudo mkdir /shared && sudo chown :team /shared
  sudo chmod 2775 /shared      → SGID

AUDIT:
  getent group sudo             → Who has sudo?
  find / -group grp 2>/dev/null → Files by group
```

---

> **Previous**: [`su/sudo` ←](./25_su_sudo.md) | **Next**: [`ps` →](../04_process_management/27_ps.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
