# 🛠️ `uniq` — Report or Filter Repeated Lines | Linux Master Note

> **Remove duplicates, count occurrences, find unique values. The perfect partner for `sort`.**

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

`uniq` filters **adjacent** duplicate lines from sorted input. Key word: **adjacent** — the input MUST be sorted first, otherwise `uniq` won't catch non-adjacent duplicates.

### Golden Rule:
```bash
sort file.txt | uniq      # ✅ CORRECT — always sort first!
uniq file.txt              # ❌ WRONG — misses non-adjacent duplicates
```

---

## 🧰 Syntax & Options

```bash
uniq [OPTIONS] [INPUT [OUTPUT]]
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-c` | `--count` | Prefix lines with occurrence count |
| `-d` | `--repeated` | Only print duplicate lines |
| `-D` | `--all-repeated` | Print ALL duplicate lines |
| `-u` | `--unique` | Only print unique (non-repeated) lines |
| `-i` | `--ignore-case` | Case-insensitive comparison |
| `-f NUM` | `--skip-fields=NUM` | Skip first NUM fields |
| `-s NUM` | `--skip-chars=NUM` | Skip first NUM characters |
| `-w NUM` | `--check-chars=NUM` | Compare only first NUM characters |

---

## 🟢 Basic Usage

```bash
# Remove adjacent duplicates (SORT FIRST!)
$ sort file.txt | uniq

# Count occurrences
$ sort file.txt | uniq -c
      3 apple
      1 banana
      5 cherry

# Show only duplicated lines
$ sort file.txt | uniq -d

# Show only unique (non-duplicated) lines
$ sort file.txt | uniq -u
```

---

## 🟡 Intermediate Usage

### Count and sort by frequency
```bash
# Most common values (top 10)
$ sort data.txt | uniq -c | sort -rn | head -10
    847 error
    234 warning
     42 critical

# Least common values
$ sort data.txt | uniq -c | sort -n | head -10
```

### Case-insensitive
```bash
$ sort -f data.txt | uniq -ic
      5 ERROR     # Counts ERROR, error, Error as same
```

### Skip fields for comparison
```bash
# Skip first field (compare from 2nd field onwards)
$ sort -k2 log.txt | uniq -f1
# Useful when first field is timestamp
```

### Compare only first N characters
```bash
$ sort file.txt | uniq -w 5    # Compare only first 5 chars
```

---

## 🔴 Advanced Usage

### Log analysis — top IPs
```bash
$ awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
    1247 192.168.1.100
     834 10.0.0.55
     456 172.16.0.1
```

### Find duplicate files by hash
```bash
$ find . -type f -exec md5sum {} + | sort | uniq -d -w 32
```

### Security — detect brute force
```bash
# IPs with most failed SSH attempts
$ grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -5

# Users targeted in brute force
$ grep "Failed password" /var/log/auth.log | awk '{print $(NF-5)}' | sort | uniq -c | sort -rn
```

### Print ALL duplicate lines (not just one)
```bash
$ sort file.txt | uniq -D
apple
apple
apple
cherry
cherry
```

---

## 🔗 Piping & Combining

```bash
# Classic frequency analysis pipeline
$ cat file.txt | sort | uniq -c | sort -rn

# Word frequency in a document
$ tr ' ' '\n' < document.txt | tr 'A-Z' 'a-z' | sort | uniq -c | sort -rn | head -20

# HTTP status code analysis
$ awk '{print $9}' access.log | sort | uniq -c | sort -rn
   5432 200
   1234 404
    567 500

# Find duplicate lines between two files
$ sort file1.txt file2.txt | uniq -d

# Find lines unique to each file
$ sort file1.txt file2.txt | uniq -u
```

---

## 💡 Real World Pro Tips

1. **ALWAYS `sort` before `uniq`** — `uniq` only detects adjacent duplicates
2. **`sort -u`** is faster than `sort | uniq` for simple dedup
3. **`sort | uniq -c | sort -rn`** = the ultimate frequency analysis pipeline
4. **Use `awk '!seen[$0]++'`** to deduplicate without sorting (preserves order)
5. **`uniq -c`** is essential for log analysis and security monitoring

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Simple frequency counting | MUST sort input first |
| Duplicate detection | Only compares adjacent lines |
| Unique line extraction | Limited comparison options |
| Field/char skip options | No regex support |

---

## 📍 Where & When to Use

| Scenario | Use `uniq`? | Alternative |
|----------|-----------|-------------|
| Remove duplicates | ✅ `sort \| uniq` | `sort -u` |
| Count occurrences | ✅ `uniq -c` | `awk` associative arrays |
| Preserve order dedup | ❌ No | `awk '!seen[$0]++'` |
| Find duplicates | ✅ `uniq -d` | — |
| Complex dedup logic | ❌ Limited | `awk`, Python |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Remove duplicate lines from a file
2. Count occurrences of each line

### 🟡 Intermediate
3. Find the top 10 most common values
4. Show only lines that appear exactly once
5. Case-insensitive deduplication

### 🔴 Advanced
6. Analyze access log — top 10 IPs and status codes
7. Find brute force attackers from auth.log
8. Word frequency analysis on a text document
9. Compare two files — find common and unique lines

---

## 🧠 Cheat Sheet

```
sort file | uniq            → Remove duplicates    uniq -u  → Unique only
sort file | uniq -c         → Count occurrences    uniq -d  → Dupes only
sort file | uniq -c | sort -rn  → Frequency rank   uniq -D  → All dupes
sort file | uniq -i         → Case insensitive     uniq -f1 → Skip field
sort file | uniq -w 5       → Compare first 5 chars

⚠️ ALWAYS sort before uniq! Or use: sort -u (faster)
FREQUENCY PIPELINE: sort | uniq -c | sort -rn | head -10
```

---

> **Previous**: [`sort` ←](./18_sort.md) | **Next**: [`wc` →](./20_wc.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
