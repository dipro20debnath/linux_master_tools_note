# 🛠️ `sort` — Sort Lines of Text | Linux Master Note

> **Organize any text data — alphabetically, numerically, by columns, or by custom rules.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Syntax & Options](#-syntax--options)
3. [Basic Usage](#-basic-usage)
4. [Intermediate Usage](#-intermediate-usage)
5. [Advanced Usage](#-advanced-usage)
6. [Piping & Combining](#-piping--combining)
7. [Real World Pro Tips](#-real-world-pro-tips)
8. [Pros & Cons](#-pros--cons)
9. [Where & When to Use](#-where--when-to-use)
10. [Practice Exercises](#-practice-exercises)
11. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

`sort` sorts lines of text files. By default it sorts alphabetically (lexicographic), but supports numeric, month, human-readable sizes, version numbers, and custom key sorting.

---

## 🧰 Syntax & Options

```bash
sort [OPTIONS] [FILE(s)...]
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-n` | `--numeric-sort` | Sort numerically |
| `-r` | `--reverse` | Reverse sort order |
| `-k` | `--key=POS` | Sort by specific field/column |
| `-t` | `--field-separator` | Set field delimiter |
| `-u` | `--unique` | Remove duplicate lines |
| `-f` | `--ignore-case` | Case-insensitive sort |
| `-h` | `--human-numeric-sort` | Sort human sizes (2K, 1G) |
| `-M` | `--month-sort` | Sort by month name |
| `-V` | `--version-sort` | Natural version number sort |
| `-o` | `--output=FILE` | Write to file (can be same as input) |
| `-c` | `--check` | Check if already sorted |
| `-s` | `--stable` | Stabilize sort (preserve order of equal) |
| `-R` | `--random-sort` | Random shuffle |
| `-b` | `--ignore-leading-blanks` | Ignore leading whitespace |

---

## 🟢 Basic Usage

```bash
# Alphabetical sort
$ sort names.txt

# Reverse sort
$ sort -r names.txt

# Numeric sort
$ sort -n numbers.txt

# Remove duplicates while sorting
$ sort -u names.txt

# Case-insensitive sort
$ sort -f names.txt
```

---

## 🟡 Intermediate Usage

### Sort by specific column
```bash
# Sort by 2nd column (space-delimited)
$ sort -k2 data.txt

# Sort by 3rd column numerically
$ sort -k3 -n data.txt

# Sort by 2nd column, then 3rd column
$ sort -k2,2 -k3,3n data.txt

# Sort CSV by 3rd column
$ sort -t, -k3 -n data.csv
```

### Sort by human-readable sizes
```bash
$ du -sh /var/log/* | sort -h
4.0K    /var/log/lastlog
128K    /var/log/auth.log
2.5M    /var/log/syslog
1.2G    /var/log/journal
```

### Month sort
```bash
$ sort -M months.txt
Jan
Feb
Mar
Apr
```

### Version sort
```bash
$ echo -e "v1.2\nv1.10\nv1.9\nv2.0" | sort -V
v1.2
v1.9
v1.10
v2.0
```

### Sort and save to same file
```bash
$ sort -o file.txt file.txt    # Safe! (unlike sort file > file)
```

---

## 🔴 Advanced Usage

### Complex key sorting
```bash
# Sort /etc/passwd by UID (3rd field, : delimiter)
$ sort -t: -k3 -n /etc/passwd

# Sort by last field
$ awk '{print NF, $0}' file.txt | sort -n | cut -d' ' -f2-

# Sort IP addresses properly
$ sort -t. -k1,1n -k2,2n -k3,3n -k4,4n ips.txt
```

### Sort + uniq for frequency analysis
```bash
# Top 10 most common words
$ tr ' ' '\n' < file.txt | sort | uniq -c | sort -rn | head -10

# Top IPs in access log
$ awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

### Random shuffle
```bash
$ sort -R names.txt              # Random order
$ shuf names.txt                 # Better alternative
```

### Check if file is sorted
```bash
$ sort -c file.txt
sort: file.txt:3: disorder: banana
$ echo $?    # 1 = not sorted, 0 = sorted
```

### Parallel sort for huge files
```bash
$ sort --parallel=4 hugefile.txt    # Use 4 CPU cores
```

---

## 🔗 Piping & Combining

```bash
# sort + uniq — unique sorted values
$ cat data.txt | sort | uniq

# sort + head — top N items
$ sort -rn scores.txt | head -5

# ps + sort — top memory processes  
$ ps aux | sort -k4 -rn | head -10

# du + sort — largest directories
$ du -sh /var/* 2>/dev/null | sort -rh | head -10
```

---

## 💡 Real World Pro Tips

1. **`sort -u`** is faster than `sort | uniq`
2. **`sort -o file file`** safely sorts in-place (redirect `>` would empty the file!)
3. **`-k2,2`** sorts by field 2 ONLY; `-k2` sorts from field 2 to end of line
4. **`LC_ALL=C sort`** is faster (byte-value sorting, no locale overhead)
5. **Combine `-h`** with `du` for disk analysis

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Multiple sort types (num, month, human) | Complex key syntax |
| Multi-key sorting | Memory-intensive for huge files |
| Safe in-place with `-o` | Locale-dependent behavior |
| Unique filtering built-in | — |

---

## 📍 Where & When to Use

| Scenario | Use `sort`? | Alternative |
|----------|-----------|-------------|
| Sort lines | ✅ Yes | — |
| Sort + deduplicate | ✅ `sort -u` | — |
| Sort by column | ✅ `-k` flag | `awk` for complex |
| Disk usage analysis | ✅ `sort -rh` | — |
| Random shuffle | ⚠️ `sort -R` | `shuf` (better) |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Sort a file alphabetically and numerically
2. Sort in reverse order
3. Remove duplicates with `sort -u`

### 🟡 Intermediate
4. Sort `/etc/passwd` by UID (3rd field)
5. Sort `du` output by human-readable sizes
6. Sort CSV by specific column

### 🔴 Advanced
7. Multi-key sort (by column 2, then column 3)
8. Sort IP addresses correctly
9. Create a word frequency analysis pipeline
10. Sort and deduplicate a million-line file efficiently

---

## 🧠 Cheat Sheet

```
sort file            → Alphabetical     sort -n file    → Numeric
sort -r file         → Reverse          sort -h file    → Human sizes
sort -u file         → Unique           sort -M file    → Month names
sort -k2 file        → By 2nd field     sort -V file    → Version
sort -t, -k3n file   → CSV, 3rd col     sort -R file    → Random
sort -o file file    → Safe in-place    sort -c file    → Check sorted
sort -k2,2 -k3,3n    → Multi-key       LC_ALL=C sort   → Fast locale
```

---

> **Previous**: [`awk` ←](./17_awk.md) | **Next**: [`uniq` →](./19_uniq.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
