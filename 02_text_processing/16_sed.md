# 🛠️ `sed` — Stream Editor | Linux Master Note

> **The Linux text surgeon. Find, replace, insert, delete — all without opening a text editor. Master `sed` and you can transform ANY text data.**

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
11. [Common Mistakes](#-common-mistakes)
12. [Practice Exercises](#-practice-exercises)
13. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### What is `sed`?
`sed` (Stream Editor) reads text input line-by-line, applies editing commands, and outputs the result. It's **non-interactive** — perfect for scripting and automation.

### Key concept:
- `sed` does NOT modify the original file by default
- It reads from input, transforms, and writes to stdout
- Use `-i` flag to edit files **in-place**

---

## 📖 Theory

### How `sed` processes text:
```
Input → [Read line into Pattern Space] → [Apply commands] → [Output] → [Repeat]
```
- **Pattern Space**: Temporary buffer holding current line
- **Hold Space**: Secondary buffer for advanced operations
- Each line is processed independently (unless multi-line mode)

---

## 🧰 Syntax & Options

```bash
sed [OPTIONS] 'COMMAND' [FILE(s)...]
sed [OPTIONS] -e 'CMD1' -e 'CMD2' [FILE(s)...]
sed [OPTIONS] -f script.sed [FILE(s)...]
```

| Flag | Description |
|------|-------------|
| `-i` | Edit file in-place |
| `-i.bak` | In-place with backup (creates .bak file) |
| `-n` | Suppress auto-print (use with `p` command) |
| `-e` | Multiple commands |
| `-f` | Read commands from file |
| `-E` / `-r` | Extended regex |

### Core Commands

| Command | Description | Example |
|---------|-------------|---------|
| `s/old/new/` | Substitute first occurrence | `s/foo/bar/` |
| `s/old/new/g` | Substitute ALL occurrences | `s/foo/bar/g` |
| `s/old/new/gi` | Substitute all, case-insensitive | `s/foo/bar/gi` |
| `d` | Delete line | `3d` (delete line 3) |
| `p` | Print line | `3p` (print line 3) |
| `i\text` | Insert text BEFORE line | `3i\new line` |
| `a\text` | Append text AFTER line | `3a\new line` |
| `c\text` | Replace entire line | `3c\replacement` |
| `y/abc/xyz/` | Transliterate (like `tr`) | `y/aeiou/AEIOU/` |
| `q` | Quit processing | `10q` (quit after line 10) |

### Address Ranges

| Address | Description |
|---------|-------------|
| `5` | Line 5 only |
| `5,10` | Lines 5 through 10 |
| `$` | Last line |
| `1,5` | First 5 lines |
| `/pattern/` | Lines matching pattern |
| `/start/,/end/` | Range between patterns |
| `1~2` | Every odd line (1, 3, 5...) |
| `0~2` | Every even line (2, 4, 6...) |

---

## 🟢 Basic Usage

### Substitute (find and replace)
```bash
# Replace first occurrence on each line
$ echo "hello world hello" | sed 's/hello/hi/'
hi world hello

# Replace ALL occurrences
$ echo "hello world hello" | sed 's/hello/hi/g'
hi world hi

# Replace in file (print to stdout)
$ sed 's/old/new/g' file.txt

# Replace in file (in-place)
$ sed -i 's/old/new/g' file.txt

# In-place with backup
$ sed -i.bak 's/old/new/g' file.txt
```

### Delete lines
```bash
# Delete line 3
$ sed '3d' file.txt

# Delete lines 5-10
$ sed '5,10d' file.txt

# Delete last line
$ sed '$d' file.txt

# Delete empty lines
$ sed '/^$/d' file.txt

# Delete lines matching pattern
$ sed '/comment/d' config.txt
```

### Print specific lines
```bash
# Print only line 5
$ sed -n '5p' file.txt

# Print lines 10-20
$ sed -n '10,20p' file.txt

# Print lines matching pattern
$ sed -n '/error/p' log.txt
```

---

## 🟡 Intermediate Usage

### Insert and Append
```bash
# Insert BEFORE line 3
$ sed '3i\This is inserted before line 3' file.txt

# Append AFTER line 3
$ sed '3a\This is appended after line 3' file.txt

# Add header to file
$ sed '1i\# Configuration File' config.txt

# Add line after pattern
$ sed '/\[database\]/a\host=localhost' config.ini
```

### Multiple commands
```bash
# Using -e
$ sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt

# Using semicolons
$ sed 's/foo/bar/g; s/baz/qux/g' file.txt

# Using script file
$ cat commands.sed
s/foo/bar/g
s/baz/qux/g
/^#/d
$ sed -f commands.sed file.txt
```

### Using different delimiters
```bash
# When pattern contains /  — use different delimiter!
$ sed 's|/usr/local/bin|/opt/bin|g' file.txt
$ sed 's#http://old.com#https://new.com#g' file.txt
$ sed 's@/path/to/old@/path/to/new@g' file.txt
```

### Regex with sed
```bash
# Remove leading whitespace
$ sed 's/^[[:space:]]*//' file.txt

# Remove trailing whitespace
$ sed 's/[[:space:]]*$//' file.txt

# Remove both
$ sed 's/^[[:space:]]*//; s/[[:space:]]*$//' file.txt

# Remove HTML tags
$ sed 's/<[^>]*>//g' page.html

# Extract content between tags
$ sed -n 's/.*<title>\(.*\)<\/title>.*/\1/p' page.html
```

---

## 🔴 Advanced Usage

### Back-references (capture groups)
```bash
# Swap first and last name
$ echo "John Smith" | sed -E 's/(\w+) (\w+)/\2 \1/'
Smith John

# Add quotes around words
$ echo "hello world" | sed -E 's/(\w+)/"\1"/g'
"hello" "world"

# Duplicate a word
$ echo "hello" | sed -E 's/(.*)/\1 \1/'
hello hello

# Reformat date: MM/DD/YYYY → YYYY-MM-DD
$ echo "05/11/2026" | sed -E 's|([0-9]{2})/([0-9]{2})/([0-9]{4})|\3-\1-\2|'
2026-05-11
```

### Cybersecurity use cases 🔒
```bash
# Sanitize log files (remove IPs)
$ sed -E 's/[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}/[REDACTED]/g' access.log

# Remove comments from config (for audit)
$ sed '/^#/d; /^$/d' /etc/ssh/sshd_config

# Mass update credentials in configs
$ sed -i 's/old_password/new_password/g' /etc/app/*.conf

# Strip sensitive data before sharing
$ sed -E 's/(password|secret|key)\s*=\s*.*/\1=REDACTED/gi' config.env
```

### Multi-line operations
```bash
# Join every 2 lines
$ sed 'N;s/\n/ /' file.txt

# Replace across line breaks
$ sed ':a;N;$!ba;s/\n/ /g' file.txt   # Join ALL lines into one
```

### Conditional operations
```bash
# Replace only on lines matching pattern
$ sed '/production/s/debug=true/debug=false/' config.txt

# Replace between two patterns
$ sed '/BEGIN/,/END/s/old/new/g' file.txt
```

---

## 🔗 Piping & Combining

```bash
# sed + grep
$ grep "error" log.txt | sed 's/.*error: //'

# sed + awk
$ cat data.csv | sed 's/,/\t/g' | awk '{print $1, $3}'

# sed + find (mass edit)
$ find . -name "*.py" -exec sed -i 's/python2/python3/g' {} +

# Pipeline processing
$ cat raw.txt | sed 's/  */ /g' | sed 's/^ //' | sort | uniq
```

---

## 💡 Real World Pro Tips

### Tip 1: Always backup before in-place edit
```bash
$ sed -i.bak 's/old/new/g' important.conf
# Creates important.conf.bak before editing
```

### Tip 2: Test before applying
```bash
# Preview changes (no -i)
$ sed 's/old/new/g' file.txt | diff file.txt -
```

### Tip 3: Use `-E` for cleaner regex
```bash
# Without -E (escape everything)
$ sed 's/\([0-9]\{3\}\)/(\1)/g' file.txt

# With -E (much cleaner)
$ sed -E 's/([0-9]{3})/(\1)/g' file.txt
```

### Tip 4: Config file management
```bash
# Update a specific config value
$ sed -i 's/^port=.*/port=8080/' app.conf

# Comment out a line
$ sed -i 's/^dangerous_setting/#&/' config.txt

# Uncomment a line
$ sed -i 's/^#\(wanted_setting\)/\1/' config.txt
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Non-interactive (scriptable) | Complex syntax for beginners |
| In-place editing | Multi-line operations are hard |
| Regex support | Not great for structured data (use `awk`) |
| Pipe-friendly | Different syntax on macOS (BSD sed) |
| Universal availability | No arithmetic operations |

---

## 📍 Where & When to Use

| Scenario | Use `sed`? | Alternative |
|----------|-----------|-------------|
| Find & replace in files | ✅ Yes | `perl -pi -e` |
| Delete/insert lines | ✅ Yes | — |
| Config file editing | ✅ Yes | — |
| Structured data (CSV/JSON) | ❌ No | `awk`, `jq` |
| Complex text processing | ⚠️ Limited | `awk`, `perl` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `-i` for in-place | Add `-i` (or `-i.bak` for safety) |
| `/` in pattern breaks command | Use different delimiter: `s\|old\|new\|` |
| Greedy matching | Use `[^>]*` instead of `.*` |
| BSD vs GNU sed differences | Use `sed -i '' 's/...'` on macOS |
| Not testing before `-i` | Always preview without `-i` first |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Replace "hello" with "hi" in a file
2. Delete all empty lines
3. Print only lines 5-10

### 🟡 Intermediate
4. Replace text in-place with backup
5. Remove all HTML tags from a file
6. Add a header line to a file
7. Use different delimiters for URL replacement

### 🔴 Advanced
8. Swap first/last name using capture groups
9. Reformat dates from MM/DD/YYYY to YYYY-MM-DD
10. Strip sensitive data from config files
11. Mass-edit all `.py` files using `find + sed`
12. Comment/uncomment config lines programmatically

---

## 🧠 Cheat Sheet

```
sed 's/old/new/' file       → Replace first         sed '3d' file    → Delete line 3
sed 's/old/new/g' file      → Replace all           sed '/pat/d'     → Delete matching
sed -i 's/old/new/g' file   → In-place edit         sed -n '5p'      → Print line 5
sed -i.bak 's/old/new/g'    → In-place + backup     sed -n '5,10p'   → Print range
sed -E 's/(grp)/\1/' file   → Extended regex         sed '/^$/d'      → Delete blank
sed 's|/old|/new|g'         → Alt delimiter          sed '3i\text'    → Insert before

ADDRESS: 5=line5  5,10=range  $=last  /pat/=match  /s/,/e/=between
```

---

> **Previous**: [`grep` ←](./15_grep.md) | **Next**: [`awk` →](./17_awk.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
