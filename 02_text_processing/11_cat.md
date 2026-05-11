# 🛠️ `cat` — Concatenate & Display Files | Linux Master Note

> **The simplest way to view file contents. But `cat` is far more powerful than most people think.**

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

### What is `cat`?
`cat` (concatenate) reads files sequentially and writes their contents to stdout. Despite its name suggesting concatenation, it's most commonly used to simply **display file contents**.

### History
- Created for **Unix V1** (1971) by Ken Thompson
- Part of **GNU Coreutils**
- Named "cat" because it con**cat**enates files

---

## 📖 Theory

### How `cat` works internally:
1. Opens each file argument using `open()` system call
2. Reads data in blocks using `read()`
3. Writes to stdout using `write()`
4. If no file specified, reads from **stdin** (keyboard input)

### stdin/stdout concept:
```
stdin (fd 0)  →  [cat]  →  stdout (fd 1)
keyboard/pipe           →  screen/pipe/file
```

---

## 🧰 Syntax & Options

```bash
cat [OPTIONS] [FILE(s)...]
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-n` | `--number` | Number all output lines |
| `-b` | `--number-nonblank` | Number non-blank lines only |
| `-s` | `--squeeze-blank` | Squeeze multiple blank lines into one |
| `-E` | `--show-ends` | Show `$` at end of each line |
| `-T` | `--show-tabs` | Show tabs as `^I` |
| `-A` | `--show-all` | Equivalent to `-vET` (show all invisible chars) |
| `-v` | `--show-nonprinting` | Show non-printing characters |
| `-e` | — | Equivalent to `-vE` |
| `-t` | — | Equivalent to `-vT` |

---

## 🟢 Basic Usage

### Display file contents
```bash
$ cat file.txt
Hello World
This is a text file.
```

### Display multiple files
```bash
$ cat file1.txt file2.txt file3.txt
```

### Concatenate files into one
```bash
$ cat part1.txt part2.txt part3.txt > complete.txt
```

### Create a new file (type content)
```bash
$ cat > newfile.txt
Type your content here
Press Ctrl+D to save
^D

$ cat >> existingfile.txt   # Append mode
More content appended
^D
```

---

## 🟡 Intermediate Usage

### Number lines
```bash
$ cat -n script.sh
     1	#!/bin/bash
     2	echo "Hello"
     3	
     4	echo "World"

# Number only non-blank lines
$ cat -b script.sh
     1	#!/bin/bash
     2	echo "Hello"

     3	echo "World"
```

### Show invisible characters
```bash
# Show line endings ($)
$ cat -E file.txt
Hello World$
Tab	here$

# Show tabs (^I)
$ cat -T file.txt
Hello World
Tab^Ihere

# Show ALL invisible characters
$ cat -A file.txt
Hello World$
Tab^Ihere$
```

### Squeeze blank lines
```bash
# Before
$ cat messy.txt
Line 1


 

Line 2

# After squeezing
$ cat -s messy.txt
Line 1

Line 2
```

### Here document (heredoc)
```bash
$ cat << EOF > config.txt
server=localhost
port=8080
debug=true
EOF
```

---

## 🔴 Advanced Usage

### Concatenate with line numbers for code review
```bash
$ cat -n /etc/nginx/nginx.conf | head -20
```

### Binary file detection
```bash
# cat can display binary files (messy output — use for detection)
$ cat /bin/ls | head -1
# Shows garbled output = binary file

# Better: check file type first
$ file /bin/ls
/bin/ls: ELF 64-bit LSB pie executable
```

### Create multi-line files in scripts
```bash
#!/bin/bash
cat > /etc/app/config.json << 'JSONEOF'
{
    "database": {
        "host": "localhost",
        "port": 5432
    },
    "debug": false
}
JSONEOF
```

### Useless Use of Cat (UUOC) — avoid this!
```bash
# ❌ WRONG — Useless Use of Cat
$ cat file.txt | grep "pattern"
$ cat file.txt | wc -l
$ cat file.txt | head -5

# ✅ CORRECT — Direct input
$ grep "pattern" file.txt
$ wc -l < file.txt
$ head -5 file.txt
```

### `tac` — reverse cat (print lines in reverse)
```bash
$ tac file.txt          # Last line first
$ cat file.txt | tac    # Same thing (but UUOC!)
```

### Combine files with separator
```bash
$ for f in *.log; do echo "=== $f ===" && cat "$f"; done
```

---

## 🔗 Piping & Combining

```bash
# cat + grep — Search in file content
$ cat access.log | grep "404"        # (UUOC — use grep directly)

# cat + wc — Count lines
$ cat file.txt | wc -l               # (UUOC)

# cat + sort — Sort file content
$ cat names.txt | sort

# cat + tr — Transform characters
$ cat file.txt | tr 'a-z' 'A-Z'     # To uppercase

# cat + sed — Edit stream
$ cat template.txt | sed 's/NAME/Dipro/g'

# Concatenate + redirect (actual valid use!)
$ cat header.html body.html footer.html > page.html
```

---

## 💡 Real World Pro Tips

### Tip 1: Quick file viewing alternatives
```bash
cat file.txt        # Small files only
less file.txt       # Large files (with scrolling)
head -20 file.txt   # First 20 lines
tail -20 file.txt   # Last 20 lines
```

### Tip 2: Modern alternatives
```bash
# bat — cat with syntax highlighting
$ sudo apt install bat
$ bat script.py       # Colored output with line numbers!
```

### Tip 3: Create files from command line
```bash
# Quick one-liner
$ echo "Hello World" > file.txt

# Multi-line
$ cat << 'EOF' > script.sh
#!/bin/bash
echo "Automated script"
date
EOF
```

### Tip 4: Check for Windows vs Unix line endings
```bash
$ cat -A file.txt
# Windows: lines end with ^M$ (carriage return + newline)
# Unix: lines end with $ only
# Fix: dos2unix file.txt
```

### Tip 5: Cybersecurity — examine suspicious files
```bash
# Safely view potentially malicious scripts
$ cat -A suspicious.sh    # Shows hidden characters
$ cat -v malware.txt      # Shows non-printing chars
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Simplest file viewer | Bad for large files (no scroll) |
| Concatenation power | UUOC is common anti-pattern |
| Show invisible chars | No syntax highlighting |
| Universal availability | No search/navigation |
| Here document support | Binary files produce garbage |

---

## 📍 Where & When to Use

| Scenario | Use `cat`? | Better Alternative |
|----------|-----------|-------------------|
| View small file (<50 lines) | ✅ Yes | — |
| View large file | ❌ No | `less`, `more` |
| Concatenate multiple files | ✅ Yes | — |
| Create quick files | ✅ Yes (heredoc) | `nano`, `vim` |
| Pipe to another command | ❌ Usually UUOC | Direct input |
| Check invisible chars | ✅ `cat -A` | `xxd` for hex |
| Syntax highlighted view | ❌ No | `bat` |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Viewing huge files | Use `less` or `head`/`tail` |
| `cat file \| grep` (UUOC) | `grep pattern file` directly |
| Overwriting with `>` | Use `>>` to append |
| Binary file garbage | Check with `file` command first |
| Missing heredoc EOF | EOF must be alone on its line |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Display contents of `/etc/hostname`
2. Create a file using `cat > file.txt`
3. Concatenate two files into one

### 🟡 Intermediate
4. Display a file with line numbers
5. Show invisible characters with `cat -A`
6. Squeeze multiple blank lines with `-s`
7. Use heredoc to create a config file

### 🔴 Advanced
8. Identify 3 examples of UUOC and fix them
9. Check line endings (Windows vs Unix) with `cat -A`
10. Install `bat` and compare with `cat`
11. Use `tac` to reverse a file

---

## 🧠 Cheat Sheet

```
cat file             → Display file        cat -n file  → Line numbers
cat f1 f2 > f3       → Concatenate         cat -b file  → Number non-blank
cat > file           → Create (Ctrl+D)     cat -s file  → Squeeze blanks
cat >> file          → Append              cat -A file  → Show all invisible
cat << EOF > file    → Heredoc             cat -E file  → Show line ends
tac file             → Reverse lines       cat -T file  → Show tabs

⚠️ AVOID UUOC: cat file | cmd  →  cmd < file  OR  cmd file
```

---

> **Next**: [`less/more` →](./12_less_more.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
