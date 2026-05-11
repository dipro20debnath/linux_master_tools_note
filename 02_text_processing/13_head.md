# 🛠️ `head` — View First Lines of File | Linux Master Note

> **Quickly peek at the beginning of any file. Perfect for checking file headers, CSV columns, and log structure.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [Piping & Combining](#-piping--combining)
8. [Real World Pro Tips](#-real-world-pro-tips)
9. [Pros & Cons](#-pros--cons)
10. [Where & When to Use](#-where--when-to-use)
11. [Practice Exercises](#-practice-exercises)
12. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

`head` prints the **first N lines** (default: 10) of a file. It's the counterpart of `tail`. Extremely useful for quickly inspecting file structure without loading the entire file.

---

## 🧰 Syntax & Options

```bash
head [OPTIONS] [FILE(s)...]
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-n NUM` | `--lines=NUM` | Print first NUM lines (default 10) |
| `-c NUM` | `--bytes=NUM` | Print first NUM bytes |
| `-q` | `--quiet` | Don't print filename headers |
| `-v` | `--verbose` | Always print filename headers |
| `-n -NUM` | — | Print all EXCEPT last NUM lines |
| `-c -NUM` | — | Print all EXCEPT last NUM bytes |

---

## 🟢 Basic Usage

```bash
# First 10 lines (default)
$ head /etc/passwd

# First 5 lines
$ head -n 5 /etc/passwd
$ head -5 /etc/passwd            # Shorthand

# First 100 bytes
$ head -c 100 /var/log/syslog

# Multiple files (shows headers)
$ head file1.txt file2.txt
==> file1.txt <==
(first 10 lines)

==> file2.txt <==
(first 10 lines)
```

---

## 🟡 Intermediate Usage

### Print all except last N lines
```bash
$ head -n -5 file.txt    # Everything EXCEPT last 5 lines
```

### Check CSV/data file headers
```bash
$ head -1 data.csv       # Column headers only
Name,Age,City,Email
```

### Quick file structure check
```bash
$ head -20 /etc/nginx/nginx.conf   # See config structure
```

---

## 🔴 Advanced Usage

### Combine with other tools
```bash
# Top 5 largest files
$ ls -lSh /var/log | head -6      # +1 for total line

# First line of every file in directory
$ head -1 *.csv

# Check first bytes (file magic numbers)
$ head -c 4 file.pdf | xxd
# %PDF = PDF file, PK = ZIP, ELF = binary
```

### In scripts
```bash
#!/bin/bash
# Get column headers from CSV
HEADERS=$(head -1 data.csv)
echo "Columns: $HEADERS"

# Process file without header
tail -n +2 data.csv | while read line; do
    echo "Processing: $line"
done
```

---

## 🔗 Piping & Combining

```bash
$ dmesg | head -20              # First 20 kernel messages
$ ps aux | head -11             # Header + top 10 processes
$ history | head -10            # First 10 history entries
$ find / -name "*.log" 2>/dev/null | head -5   # First 5 results
```

---

## 💡 Real World Pro Tips

1. **`head -1`** is perfect for extracting CSV headers
2. **`head -c`** for checking file type by magic bytes
3. **Pipe with `ls -lS`** to find largest files quickly
4. **`head -n -N`** (negative) removes last N lines — great for removing trailing newlines

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Instant — reads only what needed | Forward-only (no scrolling) |
| Very fast on huge files | Can't search within output |
| Simple, predictable | No interactive mode |
| Handles multiple files | — |

---

## 📍 Where & When to Use

| Scenario | Use `head`? | Alternative |
|----------|-----------|-------------|
| Quick file peek | ✅ Yes | — |
| CSV column check | ✅ `head -1` | — |
| Full file browsing | ❌ No | `less` |
| File end checking | ❌ No | `tail` |

---

## 📝 Practice Exercises

1. Show first 5 lines of `/etc/passwd`
2. Show first 50 bytes of a file
3. Show everything except the last 3 lines
4. Check CSV headers with `head -1`
5. Find top 10 largest files using `ls -lSh | head`

---

## 🧠 Cheat Sheet

```
head file         → First 10 lines      head -c 100 file → First 100 bytes
head -n 5 file    → First 5 lines       head -n -5 file  → All except last 5
head -1 *.csv     → Headers of CSVs     head -q f1 f2    → No file headers
```

---

> **Previous**: [`less/more` ←](./12_less_more.md) | **Next**: [`tail` →](./14_tail.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
