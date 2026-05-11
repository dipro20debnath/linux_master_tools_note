# 🛠️ `awk` — Pattern Scanning & Processing Language | Linux Master Note

> **`awk` is not just a command — it's a complete programming language for text processing. The KING of data extraction.**

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

### What is `awk`?
`awk` is a pattern-action language designed for processing structured text data (columns/fields). Named after its creators: **A**ho, **W**einberger, **K**ernighan.

### Variants:
- `awk` — Original
- `gawk` — GNU awk (most common on Linux, feature-rich)
- `mawk` — Fast, minimal awk
- `nawk` — New awk (Solaris)

---

## 📖 Theory

### How `awk` processes data:
```
Input → [Split into Records (lines)] → [Split into Fields ($1,$2...)] → [Match Pattern] → [Execute Action] → Output
```

### Key concepts:
- **Record** = One line of input (default separator: newline)
- **Field** = Column in a record (default separator: whitespace)
- `$0` = Entire line, `$1` = 1st field, `$2` = 2nd field, etc.
- `NR` = Record number (line number)
- `NF` = Number of fields in current record
- `FS` = Field separator, `OFS` = Output field separator

---

## 🧰 Syntax & Options

```bash
awk 'pattern { action }' file
awk -F'delimiter' 'pattern { action }' file
awk -f script.awk file
```

### Built-in Variables

| Variable | Description |
|----------|-------------|
| `$0` | Entire current line |
| `$1, $2...` | 1st, 2nd... field |
| `NR` | Current record (line) number |
| `NF` | Number of fields in current record |
| `FS` | Input field separator (default: space/tab) |
| `OFS` | Output field separator (default: space) |
| `RS` | Record separator (default: newline) |
| `ORS` | Output record separator |
| `FILENAME` | Current input filename |
| `FNR` | Record number in current file |
| `BEGIN` | Execute before processing |
| `END` | Execute after all processing |

---

## 🟢 Basic Usage

### Print specific columns
```bash
# Print 1st column
$ awk '{print $1}' file.txt

# Print 1st and 3rd columns
$ awk '{print $1, $3}' file.txt

# Print entire line
$ awk '{print $0}' file.txt

# Print last column
$ awk '{print $NF}' file.txt
```

### Using with `/etc/passwd`
```bash
# Print usernames (field 1, delimiter is :)
$ awk -F: '{print $1}' /etc/passwd

# Print username and shell
$ awk -F: '{print $1, $7}' /etc/passwd

# Print username and home directory
$ awk -F: '{print $1 " -> " $6}' /etc/passwd
```

### Pattern matching
```bash
# Print lines containing "error"
$ awk '/error/' log.txt

# Print lines where 3rd field > 100
$ awk '$3 > 100' data.txt

# Print lines with more than 5 fields
$ awk 'NF > 5' file.txt
```

---

## 🟡 Intermediate Usage

### BEGIN and END blocks
```bash
# Header and footer
$ awk 'BEGIN {print "=== Report ==="} {print $0} END {print "=== Done ==="}' file.txt

# Count lines
$ awk 'END {print "Total lines:", NR}' file.txt

# Sum a column
$ awk '{sum += $3} END {print "Total:", sum}' data.txt

# Average
$ awk '{sum += $3; count++} END {print "Average:", sum/count}' data.txt
```

### Formatted output with `printf`
```bash
$ awk -F: '{printf "%-15s %s\n", $1, $7}' /etc/passwd
root            /bin/bash
daemon          /usr/sbin/nologin
dipro           /bin/bash
```

### Custom field separator
```bash
# CSV processing
$ awk -F, '{print $1, $3}' data.csv

# Tab-separated
$ awk -F'\t' '{print $1, $2}' data.tsv

# Multiple separators
$ awk -F'[,;:]' '{print $1, $2}' mixed.txt

# Set output separator
$ awk -F: 'BEGIN{OFS=","} {print $1, $6, $7}' /etc/passwd
```

### Conditional logic
```bash
# If-else
$ awk '{if ($3 > 90) print $1, "PASS"; else print $1, "FAIL"}' grades.txt

# Ternary
$ awk '{print $1, ($3 > 90 ? "PASS" : "FAIL")}' grades.txt

# Multiple conditions
$ awk '$3 > 90 && $4 > 80 {print $1, "Excellent"}' grades.txt
```

---

## 🔴 Advanced Usage

### Associative arrays (hash maps)
```bash
# Count word frequency
$ awk '{for(i=1;i<=NF;i++) freq[$i]++} END {for(w in freq) print w, freq[w]}' file.txt

# Count occurrences of field values
$ awk -F, '{count[$2]++} END {for(c in count) print c, count[c]}' data.csv

# Group by and sum
$ awk -F, '{sum[$1] += $3} END {for(k in sum) print k, sum[k]}' sales.csv
```

### String functions
```bash
# Length
$ awk '{print length($0), $0}' file.txt

# Substring
$ awk '{print substr($1, 1, 3)}' file.txt

# To upper/lower (gawk)
$ awk '{print toupper($1)}' file.txt
$ awk '{print tolower($0)}' file.txt

# Split string
$ echo "a:b:c:d" | awk '{split($0, arr, ":"); print arr[2]}'
b

# Substitution
$ awk '{gsub(/old/, "new"); print}' file.txt

# Match and extract
$ awk 'match($0, /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/) {print substr($0, RSTART, RLENGTH)}' log.txt
```

### Cybersecurity use cases 🔒
```bash
# Analyze Apache access log — top 10 IPs
$ awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Failed SSH login analysis
$ awk '/Failed password/ {print $(NF-3)}' /var/log/auth.log | sort | uniq -c | sort -rn

# Bandwidth analysis (bytes per IP)
$ awk '{ip[$1] += $10} END {for(i in ip) print ip[i], i}' access.log | sort -rn | head -10

# HTTP status code summary
$ awk '{status[$9]++} END {for(s in status) printf "%s: %d\n", s, status[s]}' access.log

# Find slow requests (response time > 5s)
$ awk '$NF > 5 {print $7, $NF "s"}' access.log

# Extract emails from file
$ awk 'match($0, /[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/) {print substr($0, RSTART, RLENGTH)}' file.txt
```

### Multi-file processing
```bash
# Compare two files (like simple join)
$ awk 'NR==FNR {a[$1]=$2; next} $1 in a {print $1, a[$1], $2}' file1.txt file2.txt
```

### Writing awk scripts
```bash
# script.awk
BEGIN {
    FS = ","
    OFS = "\t"
    print "Name", "Score", "Grade"
    print "----", "-----", "-----"
}
{
    grade = ($3 >= 90) ? "A" : ($3 >= 80) ? "B" : ($3 >= 70) ? "C" : "F"
    printf "%-15s %5d %5s\n", $1, $3, grade
    total += $3
    count++
}
END {
    printf "\nAverage: %.2f\n", total/count
}

$ awk -f script.awk students.csv
```

---

## 🔗 Piping & Combining

```bash
# ps + awk — process memory usage
$ ps aux | awk '{mem += $6} END {print "Total Memory:", mem/1024, "MB"}'

# df + awk — disk usage summary
$ df -h | awk 'NR>1 {print $5, $6}' | sort -rn

# netstat + awk — connection counts by state
$ netstat -an | awk '/tcp/ {print $6}' | sort | uniq -c | sort -rn

# Log analysis pipeline
$ cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10
```

---

## 💡 Real World Pro Tips

### Tip 1: Quick CSV column extraction
```bash
$ awk -F, '{print $1","$3","$5}' data.csv > subset.csv
```

### Tip 2: Remove duplicate lines (preserving order)
```bash
$ awk '!seen[$0]++' file.txt
```

### Tip 3: Print between two patterns
```bash
$ awk '/START/,/END/' file.txt
```

### Tip 4: One-liner server monitoring
```bash
# Top 5 memory-consuming processes
$ ps aux | awk 'NR>1 {print $6/1024 "MB", $11}' | sort -rn | head -5
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Complete programming language | Steeper learning curve |
| Excellent for columnar data | Complex syntax for beginners |
| Associative arrays built-in | Not great for binary data |
| Printf for formatted output | Verbose for simple tasks |
| Fast and efficient | Different versions (gawk vs mawk) |

---

## 📍 Where & When to Use

| Scenario | Use `awk`? | Alternative |
|----------|-----------|-------------|
| Column extraction | ✅ Yes | `cut` (simpler) |
| Data aggregation/stats | ✅ Yes | `datamash` |
| Log analysis | ✅ Yes | — |
| Simple find/replace | ❌ Overkill | `sed` |
| JSON/XML processing | ❌ No | `jq`, `xmllint` |
| Complex data pipelines | ✅ Yes | Python, Perl |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Print usernames from `/etc/passwd`
2. Print the 2nd column of `ls -l` output
3. Print lines containing "error" in a log file

### 🟡 Intermediate
4. Calculate sum and average of a numeric column
5. Count unique values in a column (like GROUP BY)
6. Format output with `printf`
7. Process a CSV file with custom delimiter

### 🔴 Advanced
8. Analyze Apache access log — top 10 IPs by request count
9. Create a word frequency counter
10. Write an awk script for grade calculation
11. Extract and count HTTP status codes from access log
12. Deduplicate lines while preserving order

---

## 🧠 Cheat Sheet

```
awk '{print $1}' file          → 1st column       $NF = last column
awk -F: '{print $1}' file     → Custom delimiter  NR = line number
awk '/pat/' file               → Pattern match     NF = field count
awk '$3>100' file              → Condition         $0 = whole line
awk '{sum+=$1} END{print sum}' → Sum column
awk '!seen[$0]++' file         → Remove duplicates
awk 'BEGIN{OFS=","} {print $1,$2}'  → Custom output sep
awk '{printf "%-10s %5d\n",$1,$2}'  → Formatted output

BLOCKS:  BEGIN{...}  {main}  END{...}
FUNCS:   length() substr() split() gsub() toupper() tolower()
```

---

> **Previous**: [`sed` ←](./16_sed.md) | **Next**: [`sort` →](./18_sort.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
