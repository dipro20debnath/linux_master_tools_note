# 🛠️ `mkdir` — Make Directories | Linux Master Note

> **Create directories — from simple single folders to complex nested hierarchies in one command.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage-beginner)
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

### What is `mkdir`?
`mkdir` (Make Directory) creates one or more directories. It's part of GNU Coreutils and uses the `mkdir()` system call internally.

### Key concepts:
- A directory is a special file containing filename→inode mappings
- New directories get `.` (self) and `..` (parent) entries automatically
- Default permissions: `0777 - umask` (typically `0755` → `rwxr-xr-x`)

---

## 🧰 Syntax & Options

```bash
mkdir [OPTIONS] DIRECTORY_NAME(s)
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-p` | `--parents` | Create parent directories as needed (no error if exists) |
| `-m` | `--mode=MODE` | Set permissions (like chmod) |
| `-v` | `--verbose` | Print message for each created directory |
| `-Z` | `--context=CTX` | Set SELinux context |

---

## 🟢 Basic Usage (Beginner)

```bash
# Create a single directory
$ mkdir projects

# Create multiple directories
$ mkdir dir1 dir2 dir3

# Create directory in specific path
$ mkdir /home/dipro/Documents/notes
```

---

## 🟡 Intermediate Usage

### Create nested directories with `-p`
```bash
# Without -p (FAILS if parent doesn't exist)
$ mkdir /home/dipro/a/b/c
mkdir: cannot create directory: No such file or directory

# With -p (creates all parents automatically)
$ mkdir -p /home/dipro/a/b/c    # ✅ Works!

# -p also doesn't error if directory already exists
$ mkdir -p existing_dir          # No error
```

### Set permissions with `-m`
```bash
# Create with specific permissions
$ mkdir -m 700 private_dir       # rwx------
$ mkdir -m 755 public_dir        # rwxr-xr-x
$ mkdir -m 1777 shared_dir       # Sticky bit (like /tmp)
```

### Verbose output
```bash
$ mkdir -pv project/{src,bin,docs,tests}
mkdir: created directory 'project'
mkdir: created directory 'project/src'
mkdir: created directory 'project/bin'
mkdir: created directory 'project/docs'
mkdir: created directory 'project/tests'
```

---

## 🔴 Advanced Usage

### Brace expansion for project structures
```bash
# Create complete project structure in ONE command
$ mkdir -pv myapp/{src/{components,utils,styles},public/{images,fonts},tests/{unit,integration},docs,config}

# Web project scaffold
$ mkdir -pv website/{html,css,js,images/{icons,photos},assets/{fonts,videos}}

# Date-based directory structure
$ mkdir -p logs/$(date +%Y)/$(date +%m)/$(date +%d)
# Creates: logs/2026/05/11
```

### Create directories from file list
```bash
# dirs.txt contains directory names
$ cat dirs.txt | xargs mkdir -p

# OR
$ while read dir; do mkdir -p "$dir"; done < dirs.txt
```

### Temporary directory with `mktemp`
```bash
$ TMPDIR=$(mktemp -d)
$ echo $TMPDIR
/tmp/tmp.aX4k9Qm2
# Auto-generates unique temporary directory
```

### Script: Auto project scaffolding
```bash
#!/bin/bash
PROJECT=$1
mkdir -pv "$PROJECT"/{src,tests,docs,config,scripts}
touch "$PROJECT"/{README.md,.gitignore,Makefile}
touch "$PROJECT"/src/main.py
echo "# $PROJECT" > "$PROJECT/README.md"
echo "Project '$PROJECT' created successfully!"
```

---

## 🔗 Piping & Combining

```bash
# Create dirs from command output
$ seq 1 5 | xargs -I {} mkdir -p "chapter_{}"

# mkdir + cd (create and enter)
$ mkdir -p newdir && cd newdir

# Create dir with same name as files (without extension)
$ ls *.txt | sed 's/.txt//' | xargs mkdir -p
```

---

## 💡 Real World Pro Tips

1. **Always use `-p`** — prevents errors and creates parents automatically
2. **Brace expansion** `{a,b,c}` is your best friend for complex structures
3. **`mktemp -d`** for secure temp directories in scripts
4. **Combine mkdir + touch** to scaffold projects instantly
5. **Security**: Set `mkdir -m 700` for sensitive directories

### Useful alias
```bash
# Create directory and cd into it
mkcd() { mkdir -p "$1" && cd "$1"; }
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| `-p` flag is incredibly powerful | Cannot create files (use `touch`) |
| Brace expansion for bulk creation | No template/scaffold system built-in |
| Permission setting at creation | Cannot set ownership (use `chown`) |
| Verbose mode for verification | No undo (use `rmdir` or `rm -r`) |

---

## 📍 Where & When to Use

| Scenario | Recommended |
|----------|-------------|
| Project scaffolding | `mkdir -pv` + brace expansion |
| Script temp dirs | `mktemp -d` |
| Sensitive data dirs | `mkdir -m 700` |
| Deployment structures | `mkdir -p` in deploy scripts |
| Backup directories | `mkdir -p backups/$(date +%Y-%m-%d)` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgot `-p` for nested dirs | Always use `mkdir -p` |
| Wrong permissions | Use `-m` flag or `chmod` after |
| Spaces in names | Use quotes: `mkdir "My Dir"` |
| Typo creates wrong dir | Use `-v` to verify |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Create a directory called `test_project`
2. Create three directories at once: `dir1`, `dir2`, `dir3`

### 🟡 Intermediate
3. Create nested structure: `project/src/components` in one command
4. Create a directory with permissions `700`
5. Use brace expansion to create `app/{frontend,backend,database}`

### 🔴 Advanced
6. Create a date-based log directory structure
7. Write a script that scaffolds a Python project
8. Create directories from a text file list
9. Make a `mkcd` function and add to `.bashrc`

---

## 🧠 Cheat Sheet

```
mkdir dir            → Create single dir
mkdir -p a/b/c       → Create with parents
mkdir -m 700 dir     → Set permissions
mkdir -v dir         → Verbose output
mkdir -pv d/{a,b,c}  → Brace expansion
mktemp -d            → Temp directory
mkcd() { mkdir -p "$1" && cd "$1"; }  → Create & enter
```

---

> **Previous**: [`pwd` ←](./03_pwd.md) | **Next**: [`rm` →](./05_rm.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
