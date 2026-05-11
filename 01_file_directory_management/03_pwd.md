# 🛠️ `pwd` — Print Working Directory | Linux Master Note

> **Always know where you are. `pwd` tells you your exact location in the filesystem.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--how-pwd-works)
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

### What is `pwd`?
`pwd` (Print Working Directory) displays the **absolute path** of your current directory. It answers: "Where am I right now in the filesystem?"

### Why important?
- Essential for orientation in the terminal
- Critical in scripts to ensure correct file operations
- Helps debug path-related issues
- Two versions exist: **shell built-in** and `/bin/pwd` (external)

---

## 📖 Theory — How `pwd` Works

### Two methods to determine CWD:
1. **Logical (`-L`)** — Reads `$PWD` environment variable (default). Preserves symlink names.
2. **Physical (`-P`)** — Uses `getcwd()` system call to resolve the actual filesystem path, resolving all symlinks.

### Internal flow:
```
pwd -L:  Reads $PWD variable → prints it
pwd -P:  Calls getcwd() → walks up directory tree using .. → resolves real path
```

---

## 🧰 Syntax & Options

```bash
pwd [OPTION]
```

| Flag | Description |
|------|-------------|
| `-L` | Print logical path (follow symlinks — **default**) |
| `-P` | Print physical path (resolve symlinks) |
| `--help` | Display help |
| `--version` | Display version |

---

## 🟢 Basic Usage (Beginner)

```bash
# Print current directory
$ pwd
/home/dipro/Documents

# After navigating
$ cd /var/log
$ pwd
/var/log
```

---

## 🟡 Intermediate Usage

### Logical vs Physical
```bash
$ ln -s /var/log ~/mylog
$ cd ~/mylog
$ pwd          # Logical (default)
/home/dipro/mylog
$ pwd -P       # Physical (real path)
/var/log
```

### Store in variable
```bash
CURRENT_DIR=$(pwd)
echo "I am in: $CURRENT_DIR"
```

### Built-in vs external
```bash
$ type pwd           # Shell built-in
$ /bin/pwd           # External binary
$ which pwd          # Shows path of external
```

---

## 🔴 Advanced Usage

### In scripts — save and restore directory
```bash
#!/bin/bash
ORIGINAL_DIR=$(pwd)
cd /var/log
# ... do work ...
cd "$ORIGINAL_DIR"   # Return to original
```

### Compare `$PWD` vs `pwd -P`
```bash
echo "PWD variable: $PWD"
echo "Physical:     $(pwd -P)"
echo "OLDPWD:       $OLDPWD"
```

### Dynamic prompt with `pwd`
```bash
# Show current dir in prompt
export PS1='[\u@\h $(basename $(pwd))]$ '
```

---

## 🔗 Piping & Combining

```bash
# Copy current path to clipboard (Linux with xclip)
$ pwd | xclip -selection clipboard

# Use in find command
$ find $(pwd) -name "*.log"

# Log current location
$ echo "$(date): Working in $(pwd)" >> ~/activity.log
```

---

## 💡 Real World Pro Tips

1. **Use `$PWD` in scripts** instead of calling `pwd` — it's faster (no subprocess)
2. **Always use `pwd -P`** when dealing with symlinks in deployment scripts
3. **`$OLDPWD`** stores previous directory — useful for toggling
4. Use `basename $(pwd)` to get just the current folder name

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Simple and universal | Very limited functionality |
| Two modes (logical/physical) | Only shows current dir |
| Available as built-in (fast) | No formatting options |
| `$PWD` variable always available | — |

---

## 📍 Where & When to Use

| Scenario | Use `pwd`? | Alternative |
|----------|-----------|-------------|
| Check current location | ✅ Yes | `$PWD` variable |
| Script path resolution | ✅ `pwd -P` | `realpath` |
| Prompt customization | ✅ Yes | `\w` in PS1 |
| Debug path issues | ✅ Yes | `realpath`, `readlink` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Symlink confusion | Use `pwd -P` for real path |
| Assuming `pwd` = `$PWD` | They differ with symlinks |
| Not quoting in scripts | Use `"$(pwd)"` with quotes |

---

## 📝 Practice Exercises

1. Print your current directory
2. Create a symlink, cd into it, compare `pwd` vs `pwd -P`
3. Store current dir in a variable, cd away, then return
4. Write a script that logs the working directory with timestamp

---

## 🧠 Cheat Sheet

```
pwd         → Print current directory (logical)
pwd -P      → Print physical path (resolve symlinks)
pwd -L      → Print logical path (default)
$PWD        → Environment variable (same as pwd -L)
$OLDPWD     → Previous directory
```

---

> **Previous**: [`cd` ←](./02_cd.md) | **Next**: [`mkdir` →](./04_mkdir.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
