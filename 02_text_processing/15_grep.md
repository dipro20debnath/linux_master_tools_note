# 🛠️ `grep` — Search Text Using Patterns | Linux Master Note

> **The ultimate text search tool. If `find` searches for FILES, `grep` searches INSIDE files. A must-know for every Linux user, sysadmin, and hacker.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [Regular Expressions](#-regular-expressions-with-grep)
8. [Piping & Combining](#-piping--combining)
9. [Real World Pro Tips](#-real-world-pro-tips)
10. [Pros & Cons](#-pros--cons)
11. [Where & When to Use](#-where--when-to-use)
12. [Common Mistakes](#-common-mistakes)
13. [Practice Exercises](#-practice-exercises)
14. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### What is `grep`?
`grep` (**G**lobal **R**egular **E**xpression **P**rint) searches text files for lines matching a pattern and prints them. It's one of the **top 3 most important Linux tools** (alongside `find` and `awk`).

### Variants:
- **`grep`** — Basic regular expressions (BRE)
- **`egrep`** / `grep -E` — Extended regular expressions (ERE)
- **`fgrep`** / `grep -F` — Fixed strings (no regex, fastest)
- **`rgrep`** / `grep -r` — Recursive search

---

## 📖 Theory

### How `grep` works:
1. Opens file(s) or reads from stdin
2. Reads line by line
3. Compiles the pattern into a **finite automaton** (regex engine)
4. Tests each line against the pattern
5. Prints matching lines (default) or performs specified action
6. Returns exit code: `0` = match found, `1` = no match, `2` = error

---

## 🧰 Syntax & Options

```bash
grep [OPTIONS] PATTERN [FILE(s)...]
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-i` | `--ignore-case` | Case-insensitive |
| `-v` | `--invert-match` | Show lines NOT matching |
| `-c` | `--count` | Count matching lines |
| `-n` | `--line-number` | Show line numbers |
| `-l` | `--files-with-matches` | Show only filenames |
| `-L` | `--files-without-match` | Show files NOT matching |
| `-r` / `-R` | `--recursive` | Search directories recursively |
| `-w` | `--word-regexp` | Match whole words only |
| `-x` | `--line-regexp` | Match whole lines only |
| `-o` | `--only-matching` | Print only matched part |
| `-A NUM` | `--after-context=NUM` | Show NUM lines after match |
| `-B NUM` | `--before-context=NUM` | Show NUM lines before match |
| `-C NUM` | `--context=NUM` | Show NUM lines around match |
| `-E` | `--extended-regexp` | Extended regex (egrep) |
| `-F` | `--fixed-strings` | Literal strings (fgrep) |
| `-P` | `--perl-regexp` | Perl-compatible regex (PCRE) |
| `-e` | `--regexp=PATTERN` | Multiple patterns |
| `-f` | `--file=FILE` | Read patterns from file |
| `--color` | `--color=auto` | Highlight matches |
| `--include` | — | Include file pattern |
| `--exclude` | — | Exclude file pattern |
| `--exclude-dir` | — | Exclude directories |
| `-m NUM` | `--max-count=NUM` | Stop after NUM matches |
| `-q` | `--quiet` | Quiet mode (exit code only) |
| `-H` | `--with-filename` | Print filename (default for multiple files) |
| `-h` | `--no-filename` | Don't print filename |

---

## 🟢 Basic Usage

```bash
# Search for pattern in file
$ grep "error" /var/log/syslog

# Case-insensitive
$ grep -i "error" /var/log/syslog

# Show line numbers
$ grep -n "root" /etc/passwd
1:root:x:0:0:root:/root:/bin/bash

# Count matches
$ grep -c "error" logfile.txt
42

# Invert match (lines NOT containing pattern)
$ grep -v "comment" config.txt
```

---

## 🟡 Intermediate Usage

### Context lines
```bash
# 3 lines after match
$ grep -A 3 "ERROR" /var/log/syslog

# 2 lines before match
$ grep -B 2 "CRITICAL" /var/log/syslog

# 2 lines before AND after (context)
$ grep -C 2 "Exception" app.log
```

### Recursive search
```bash
# Search in all files recursively
$ grep -r "TODO" /home/dipro/projects/

# With line numbers and filenames
$ grep -rn "import flask" /home/dipro/projects/

# Include only .py files
$ grep -rn --include="*.py" "def main" ~/projects/

# Exclude directories
$ grep -rn --exclude-dir={node_modules,.git,vendor} "api_key" ~/projects/
```

### Multiple patterns
```bash
# OR — match either pattern
$ grep -E "error|warning|critical" /var/log/syslog
$ grep -e "error" -e "warning" /var/log/syslog

# AND — match ALL patterns (use multiple greps)
$ grep "error" log.txt | grep "database" | grep "timeout"
```

### Match whole words
```bash
$ grep -w "log" file.txt     # Matches "log" but NOT "logging" or "blog"
```

### Show only matching part
```bash
$ grep -o "[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+" access.log
# Extracts only IP addresses
```

### Show only filenames
```bash
$ grep -rl "password" /etc/     # Files containing "password"
$ grep -rL "password" /etc/     # Files NOT containing "password"
```

---

## 🔴 Advanced Usage

### Perl-Compatible Regex (PCRE) — `-P`
```bash
# Lookahead: lines with "error" followed by a number
$ grep -P "error(?=.*\d)" log.txt

# Lookbehind: extract value after "port="
$ grep -oP "(?<=port=)\d+" config.txt

# Non-greedy matching
$ grep -oP "\".*?\"" file.json

# Named groups
$ echo "2026-05-11" | grep -oP "(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})"
```

### Security & Cybersecurity use cases 🔒
```bash
# Find hardcoded passwords in source code
$ grep -rn --include="*.{py,js,php,java}" -i "password\s*=" ~/projects/

# Find API keys/tokens
$ grep -rn --include="*.{py,js,env}" -E "(api_key|secret|token)\s*=" ~/projects/

# Check for SQL injection patterns
$ grep -rn --include="*.php" "SELECT.*\$_" /var/www/

# Find failed SSH logins
$ grep "Failed password" /var/log/auth.log

# Extract attacker IPs from auth log
$ grep "Failed password" /var/log/auth.log | grep -oP "\d+\.\d+\.\d+\.\d+" | sort | uniq -c | sort -rn

# Check for suspicious cron jobs
$ grep -r "curl\|wget\|nc\|netcat" /var/spool/cron/

# Find SUID binaries mentioned in configs
$ grep -rn "setuid\|suid\|chmod.*4[0-7]\{3\}" /etc/
```

### Binary file search
```bash
$ grep -r --binary-files=text "password" /opt/app/
```

### Quiet mode (for scripts)
```bash
#!/bin/bash
if grep -q "error" /var/log/app.log; then
    echo "Errors found!"
    # Send alert
fi
```

---

## 📐 Regular Expressions with grep

### BRE (Basic — default `grep`)
```
.         Any single character
*         Zero or more of previous char
^         Start of line
$         End of line
[abc]     Character class
[^abc]    Negated character class
\{n\}     Exactly n repetitions
\{n,m\}   Between n and m repetitions
\( \)     Grouping
\|        OR (GNU extension)
```

### ERE (Extended — `grep -E` or `egrep`)
```
+         One or more of previous
?         Zero or one of previous
{n}       Exactly n repetitions
{n,m}     Between n and m
( )       Grouping (no backslash needed)
|         OR (alternation)
```

### Practical regex examples
```bash
# Match email addresses
$ grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt

# Match IP addresses
$ grep -E "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" access.log

# Match URLs
$ grep -E "https?://[a-zA-Z0-9./?=_-]+" file.txt

# Match dates (YYYY-MM-DD)
$ grep -E "[0-9]{4}-[0-9]{2}-[0-9]{2}" log.txt

# Match phone numbers
$ grep -E "\+?[0-9]{1,3}[-. ]?\(?[0-9]{3}\)?[-. ]?[0-9]{3}[-. ]?[0-9]{4}" contacts.txt
```

---

## 🔗 Piping & Combining

```bash
# Process list filtering
$ ps aux | grep "nginx"
$ ps aux | grep "[n]ginx"         # Exclude grep itself from results!

# Log analysis
$ cat access.log | grep "404" | awk '{print $7}' | sort | uniq -c | sort -rn

# Network connections
$ netstat -tuln | grep "LISTEN"
$ ss -tuln | grep ":80"

# Find and search inside files
$ find . -name "*.py" -exec grep -l "import os" {} +

# History search
$ history | grep "ssh"
```

---

## 💡 Real World Pro Tips

### Tip 1: Exclude grep from ps output
```bash
# ❌ Shows grep process itself
$ ps aux | grep nginx
dipro  1234  nginx: master
dipro  5678  grep --color=auto nginx

# ✅ Character class trick
$ ps aux | grep "[n]ginx"
dipro  1234  nginx: master
```

### Tip 2: Use `ripgrep` (`rg`) for 10x speed
```bash
$ sudo apt install ripgrep
$ rg "pattern" ~/projects/      # 10x faster than grep -r
$ rg -i "TODO" --type py        # Search only Python files
```

### Tip 3: Color always in aliases
```bash
alias grep='grep --color=auto'
alias egrep='egrep --color=auto'
```

### Tip 4: Search compressed files
```bash
$ zgrep "error" /var/log/syslog.2.gz    # Search gzipped files
$ bzgrep "error" file.bz2               # Search bzip2 files
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Incredibly versatile | Regex syntax can be complex |
| Fast for most use cases | Slow for huge codebases (use `rg`) |
| Regex support (BRE/ERE/PCRE) | Binary files need special handling |
| Context lines (-A/-B/-C) | No built-in replace (use `sed`) |
| Universal availability | grep vs egrep confusion |

---

## 📍 Where & When to Use

| Scenario | Use `grep`? | Alternative |
|----------|-----------|-------------|
| Search inside files | ✅ Yes | `ripgrep` (faster) |
| Log analysis | ✅ Yes | `awk` for complex parsing |
| Pattern matching | ✅ Yes | — |
| Search + replace | ❌ No | `sed`, `awk` |
| Large codebase search | ⚠️ Slow | `ripgrep`, `ag` |
| Binary file search | ⚠️ With flags | `strings` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Not quoting pattern | Use `"pattern"` to prevent shell expansion |
| grep shows itself in ps | Use `grep "[p]attern"` trick |
| No results in binary files | Use `--binary-files=text` |
| Missing ERE features | Use `grep -E` for `+`, `?`, `\|` |
| Slow recursive search | Use `--include` filter or `ripgrep` |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Search for "root" in `/etc/passwd`
2. Case-insensitive search for "error" in a log file
3. Count how many lines contain "warning" in a log

### 🟡 Intermediate
4. Show 3 lines of context around matches
5. Recursively search for "TODO" in a project (exclude `.git`)
6. Extract all IP addresses from an access log
7. Use multiple patterns with `-e`

### 🔴 Advanced
8. Find hardcoded passwords in source code
9. Extract attacker IPs from auth log (with count)
10. Use PCRE lookahead/lookbehind to extract values
11. Install and compare `ripgrep` vs `grep` speed
12. Write a script that alerts on specific log patterns

---

## 🧠 Cheat Sheet

```
grep "pat" file         → Basic search       grep -c "pat" file → Count
grep -i "pat" file      → Case insensitive   grep -n "pat" file → Line nums
grep -v "pat" file      → Invert match       grep -l "pat" *    → Filenames only
grep -r "pat" dir/      → Recursive          grep -o "pat" file → Match only
grep -E "a|b" file      → Extended regex     grep -w "pat" file → Whole word
grep -A3 "pat" file     → 3 lines after      grep -P "pcre" file → Perl regex
grep -B3 "pat" file     → 3 lines before     grep -q "pat" file → Quiet (scripts)
grep --include="*.py" -r "pat" dir/  → Filter file types

REGEX: .=any  *=0+  +=1+  ?=0/1  ^=start  $=end  []=class  |=or
```

---

> **Previous**: [`tail` ←](./14_tail.md) | **Next**: [`sed` →](./16_sed.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
