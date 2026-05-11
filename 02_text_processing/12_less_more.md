# 🛠️ `less` & `more` — Page Through Text Files | Linux Master Note

> **Navigate large files interactively. `less` is MORE than `more` — literally.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [Navigation Keys](#-navigation-keys)
8. [Real World Pro Tips](#-real-world-pro-tips)
9. [Pros & Cons](#-pros--cons)
10. [Where & When to Use](#-where--when-to-use)
11. [Common Mistakes](#-common-mistakes)
12. [Practice Exercises](#-practice-exercises)
13. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### `more` (older)
- Forward-only pager — can only scroll **down**
- Part of original Unix (1978)
- Exits automatically at end of file

### `less` (newer, better)
- Bi-directional — scroll **up AND down**
- Name is a pun: "less is more"
- Doesn't load entire file into memory — handles **huge files**
- Supports search, regex, marks, and more
- **Always prefer `less` over `more`**

---

## 📖 Theory

### How `less` works:
- Reads file **on demand** (doesn't load entire file into memory)
- Uses terminal control sequences to manage display
- Keeps a buffer of read pages for backward navigation
- Can handle files larger than available RAM

### How `more` works:
- Loads file progressively, page by page
- Forward-only — once you scroll past, can't go back
- Simpler but far more limited

---

## 🧰 Syntax & Options

```bash
less [OPTIONS] FILE
more [OPTIONS] FILE
```

### `less` options:

| Flag | Description |
|------|-------------|
| `-N` | Show line numbers |
| `-S` | Don't wrap long lines (horizontal scroll) |
| `-i` | Case-insensitive search |
| `-I` | Case-insensitive search (even for uppercase patterns) |
| `-F` | Quit if file fits on one screen |
| `-R` | Show ANSI color escape sequences |
| `-X` | Don't clear screen on exit |
| `-M` | Show more verbose prompt (line numbers, percentage) |
| `-m` | Show percentage in prompt |
| `+F` | Follow mode (like `tail -f`) |
| `-n` | No line numbers (faster for huge files) |
| `+/pattern` | Start at first match of pattern |
| `+G` | Start at end of file |

---

## 🟢 Basic Usage

```bash
# View file with less
$ less /var/log/syslog

# View file with more
$ more /etc/passwd

# View command output
$ ls -la /etc | less
$ dmesg | less
```

---

## 🟡 Intermediate Usage

### Line numbers
```bash
$ less -N /etc/nginx/nginx.conf
      1 user www-data;
      2 worker_processes auto;
      3 pid /run/nginx.pid;
```

### Don't wrap long lines
```bash
$ less -S /var/log/access.log    # Use arrow keys to scroll horizontally
```

### Case-insensitive search
```bash
$ less -i /var/log/syslog
# Now /error will match ERROR, Error, error
```

### Show colors (for colored output)
```bash
$ ls --color=always | less -R
$ grep --color=always "error" /var/log/syslog | less -R
```

### View multiple files
```bash
$ less file1.txt file2.txt file3.txt
# :n → Next file
# :p → Previous file
# :e filename → Open another file
```

### Start at specific position
```bash
$ less +100 file.txt            # Start at line 100
$ less +/ERROR /var/log/syslog  # Start at first "ERROR"
$ less +G /var/log/syslog       # Start at end of file
```

---

## 🔴 Advanced Usage

### Follow mode (like tail -f)
```bash
$ less +F /var/log/syslog
# Shows new lines as they're added (live log monitoring)
# Press Ctrl+C to stop following, then navigate freely
# Press Shift+F to resume following
```

### Mark positions (bookmarks)
```bash
# Inside less:
ma        # Set mark 'a' at current position
mb        # Set mark 'b'
'a        # Jump to mark 'a'
'b        # Jump to mark 'b'
```

### Search with regex
```bash
# Inside less:
/error              # Search forward for "error"
?error              # Search backward for "error"
/[0-9]{3}\.[0-9]    # Regex: match IP-like patterns
n                   # Next match
N                   # Previous match
&pattern            # Show ONLY lines matching pattern
```

### Filter lines (show only matching)
```bash
# Inside less, press & then type pattern:
&ERROR              # Show only lines containing "ERROR"
&                   # Clear filter (show all lines again)
```

### Pipe while in less
```bash
# Inside less:
|$ wc -l            # Pipe current file to wc -l
```

### Environment variable customization
```bash
# Add to ~/.bashrc
export LESS='-R -i -M -S --mouse'
export LESSOPEN='|batcat --color=always %s'  # Syntax highlighting!
```

---

## 🧭 Navigation Keys

### `less` Navigation (ESSENTIAL!)

| Key | Action |
|-----|--------|
| `Space` / `f` / `Page Down` | Forward one page |
| `b` / `Page Up` | Backward one page |
| `d` | Forward half page |
| `u` | Backward half page |
| `j` / `↓` / `Enter` | Forward one line |
| `k` / `↑` | Backward one line |
| `g` / `Home` | Go to beginning of file |
| `G` / `End` | Go to end of file |
| `100g` | Go to line 100 |
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Next search match |
| `N` | Previous search match |
| `&pattern` | Show only matching lines |
| `q` | Quit |
| `h` | Help |
| `v` | Open in editor ($VISUAL or $EDITOR) |
| `F` | Follow mode (like tail -f) |
| `ma` | Set mark 'a' |
| `'a` | Go to mark 'a' |
| `:n` | Next file |
| `:p` | Previous file |
| `=` | Show file info (line count, byte position) |

### `more` Navigation (Limited)

| Key | Action |
|-----|--------|
| `Space` | Forward one page |
| `Enter` | Forward one line |
| `b` | Backward one page (some versions) |
| `/pattern` | Search forward |
| `q` | Quit |
| `=` | Show current line number |

---

## 💡 Real World Pro Tips

### Tip 1: Set less as default pager
```bash
export PAGER='less'
export MANPAGER='less -R'    # For man pages with colors
```

### Tip 2: Syntax highlighting with source-highlight
```bash
$ sudo apt install source-highlight
$ export LESSOPEN="| src-hilite-lesspipe.sh %s"
$ less script.py    # Now has syntax highlighting!
```

### Tip 3: Log analysis workflow
```bash
$ less +F /var/log/auth.log     # Follow live
# Ctrl+C → stop following
# /Failed → search for failed logins
# &Failed → filter only failed lines
# n/N → navigate matches
# F → resume following
```

### Tip 4: Reading compressed files
```bash
$ zless file.gz       # Read gzipped files directly
$ bzless file.bz2     # Read bzip2 files
$ xzless file.xz      # Read xz files
```

---

## ✅ Pros & Cons

### `less`
| ✅ Pros | ❌ Cons |
|---------|---------|
| Bi-directional scrolling | Slightly more complex than `cat` |
| Handles huge files (doesn't load all) | Not for file editing |
| Powerful search with regex | Requires learning keybindings |
| Follow mode (live logs) | — |
| Bookmark/mark system | — |
| Line filtering | — |

### `more`
| ✅ Pros | ❌ Cons |
|---------|---------|
| Very simple | Forward-only |
| Available everywhere | No regex search |
| — | No marks/bookmarks |
| — | Limited features |

---

## 📍 Where & When to Use

| Scenario | Tool | Why |
|----------|------|-----|
| View small files | `cat` | Simpler |
| View large files | `less` | Memory efficient |
| Live log monitoring | `less +F` or `tail -f` | Real-time |
| Quick file check | `head`/`tail` | Faster |
| Pipe long output | `cmd \| less` | Scrollable |
| Man pages | `less` (default) | Navigation |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `more` instead of `less` | Always prefer `less` |
| Can't scroll up | You're in `more`; use `less` |
| Colored output missing | Use `less -R` |
| Can't find search results | Check case; use `-i` flag |
| Stuck in follow mode | Press `Ctrl+C` to stop |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. View `/etc/passwd` with `less`
2. Navigate to the end and back to beginning
3. Search for "root" in the file

### 🟡 Intermediate
4. View a log file with line numbers (`-N`)
5. Open multiple files and switch between them
6. Use `&pattern` to filter lines

### 🔴 Advanced
7. Monitor a live log with `less +F`
8. Set up syntax highlighting for `less`
9. Use marks to bookmark positions in a long file
10. Configure `$LESS` environment variable in `.bashrc`

---

## 🧠 Cheat Sheet

```
OPEN:    less file | less -N file | less +F file | cmd | less
MOVE:    Space=page↓  b=page↑  g=top  G=bottom  100g=line100
SEARCH:  /pattern=fwd  ?pattern=back  n=next  N=prev
FILTER:  &pattern=show matching only  &=clear filter
MARKS:   ma=set mark  'a=goto mark
FILES:   :n=next file  :p=prev file  :e=open file
OTHER:   F=follow  v=edit  q=quit  h=help  -N=toggle numbers

less vs more:  less = scroll both ways, search, marks, follow
               more = forward only, basic
```

---

> **Previous**: [`cat` ←](./11_cat.md) | **Next**: [`head` →](./13_head.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
