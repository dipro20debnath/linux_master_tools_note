# 🛠️ `zip` & `unzip` — Cross-Platform Compression | Linux Master Note

> **The universal compression format. `zip` works everywhere — Windows, macOS, Linux — making it the go-to for sharing files across platforms.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--zip-vs-tar)
3. [Syntax & Options](#-syntax--options)
4. [Basic Usage](#-basic-usage)
5. [Intermediate Usage](#-intermediate-usage)
6. [Advanced Usage](#-advanced-usage)
7. [Real World Pro Tips](#-real-world-pro-tips)
8. [Pros & Cons](#-pros--cons)
9. [Where & When to Use](#-where--when-to-use)
10. [Common Mistakes](#-common-mistakes)
11. [Practice Exercises](#-practice-exercises)
12. [Cheat Sheet](#-cheat-sheet)

---

## 🔰 Introduction

### `zip` vs `tar.gz`:
| Feature | `zip` | `tar.gz` |
|---------|-------|---------|
| Cross-platform | ✅ Native on Windows/Mac | ⚠️ Needs tools on Windows |
| Individual file access | ✅ Extract single files | ❌ Must decompress all |
| Permission preservation | ❌ No Unix permissions | ✅ Full preservation |
| Compression | Per-file (DEFLATE) | Entire archive |
| Password protection | ✅ Built-in | ❌ Need gpg/openssl |
| Use case | **Sharing** across OS | **Backups** on Linux |

> 🎯 Use **zip** for cross-platform sharing. Use **tar.gz** for Linux backups.

---

## 🧰 Syntax & Options

### `zip`:
```bash
zip [OPTIONS] ARCHIVE.zip FILES...
```

| Flag | Description |
|------|-------------|
| `-r` | **Recursive** (include directories) |
| `-e` | **Encrypt** with password |
| `-P PASS` | Password (⚠️ visible in history!) |
| `-9` | Best compression |
| `-0` | Store only (no compression) |
| `-j` | Junk paths (don't store directory names) |
| `-x PATTERN` | Exclude files |
| `-u` | Update (add newer files only) |
| `-d` | Delete files from archive |
| `-m` | Move files into archive (delete originals) |
| `-q` | Quiet |
| `-v` | Verbose |
| `-T` | Test integrity |
| `-s SIZE` | Split archive into parts |

### `unzip`:
```bash
unzip [OPTIONS] ARCHIVE.zip
```

| Flag | Description |
|------|-------------|
| `-l` | **List** contents |
| `-t` | **Test** integrity |
| `-d DIR` | Extract to **directory** |
| `-o` | Overwrite without prompting |
| `-n` | Never overwrite |
| `-j` | Junk paths (flat extract) |
| `-p` | Extract to stdout (pipe) |
| `-P PASS` | Password |
| `-v` | Verbose listing |
| `-x PATTERN` | Exclude files during extraction |

---

## 🟢 Basic Usage

### Create zip
```bash
# Zip files
$ zip archive.zip file1.txt file2.txt

# Zip directory (MUST use -r!)
$ zip -r project.zip project/

# Zip with best compression
$ zip -r9 project.zip project/

# Zip silently
$ zip -rq project.zip project/
```

### Extract zip
```bash
# Extract to current directory
$ unzip archive.zip

# Extract to specific directory
$ unzip archive.zip -d /opt/apps/

# Extract silently, overwrite
$ unzip -oq archive.zip -d /opt/

# Extract specific files
$ unzip archive.zip "path/to/file.txt"
$ unzip archive.zip "*.conf"
```

### List contents
```bash
$ unzip -l archive.zip
Archive:  archive.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
     1234  2026-05-26 01:00   file1.txt
     5678  2026-05-26 01:00   dir/file2.txt
---------                     -------
     6912                     2 files

# Verbose listing
$ unzip -v archive.zip
```

### Test integrity
```bash
$ unzip -t archive.zip
    testing: file1.txt        OK
    testing: dir/file2.txt    OK
No errors detected in compressed data of archive.zip.

$ zip -T archive.zip
test of archive.zip OK
```

---

## 🟡 Intermediate Usage

### Password protection
```bash
# Encrypt with password (prompts)
$ zip -re secure.zip sensitive_files/

# With inline password (⚠️ visible in history!)
$ zip -rP "MyS3cretPass" secure.zip files/

# Decrypt
$ unzip secure.zip
# → Prompts for password

# ⚠️ zip encryption is WEAK (ZipCrypto)!
# Use 7z or gpg for real security
```

### Exclude files
```bash
# Exclude patterns
$ zip -r project.zip project/ -x "*.git*" -x "*/node_modules/*" -x "*.log"

# Exclude from file
$ zip -r project.zip project/ -x@exclude.lst
```

### Update and modify
```bash
# Add/update files (only if newer)
$ zip -u archive.zip newfile.txt

# Delete file from archive
$ zip -d archive.zip "old_file.txt"

# Freshen (update existing files only, don't add new)
$ zip -f archive.zip
```

### Split large archives
```bash
# Split into 100MB parts
$ zip -r -s 100m large_archive.zip large_directory/
# Creates: large_archive.z01, large_archive.z02, ..., large_archive.zip

# Combine and extract
$ cat large_archive.z* > combined.zip
$ unzip combined.zip
```

---

## 🔴 Advanced Usage

### CTF/Pentesting — Zip Cracking 🎯
```bash
# Crack zip password with fcrackzip
$ fcrackzip -u -D -p wordlist.txt encrypted.zip
PASSWORD FOUND: MyPassword

# Crack with John the Ripper
$ zip2john encrypted.zip > hash.txt
$ john hash.txt --wordlist=rockyou.txt

# Crack with hashcat
$ zip2john encrypted.zip > hash.txt
$ hashcat -m 13600 hash.txt wordlist.txt

# Known-plaintext attack (if you have one unencrypted file)
$ pkcrack -C encrypted.zip -c known_file.txt -P plain.zip -p known_file.txt -d decrypted.zip
```

### Zip bomb detection 🔒
```bash
# Check for zip bombs (tiny file → expands to TB)
$ unzip -l suspicious.zip | tail -1
42.zip:     4,503,599,627,370,496 bytes   # ⚠️ 4.5 PETABYTES!

# Safe check: list first, check sizes
$ unzip -l suspicious.zip | awk 'NR>3 && /^-/ {total+=$1} END {printf "Expands to: %.2f GB\n", total/1073741824}'

# Never extract unknown zips without checking!
```

### Batch operations
```bash
# Zip each directory separately
$ for dir in */; do
    zip -r "${dir%/}.zip" "$dir"
done

# Extract all zips in current directory
$ for f in *.zip; do
    unzip -q "$f" -d "${f%.zip}/"
done

# Find and zip old files
$ find /var/log -name "*.log" -mtime +30 | zip -@ old_logs.zip
```

### Convert between formats
```bash
# zip → tar.gz
$ unzip archive.zip -d /tmp/contents/
$ tar -czvf archive.tar.gz -C /tmp/contents/ .

# tar.gz → zip
$ tar -xzvf archive.tar.gz -C /tmp/contents/
$ cd /tmp/contents && zip -r /output/archive.zip .
```

---

## 💡 Real World Pro Tips

### Tip 1: Always use `-r` for directories
```bash
# ❌ Creates empty zip!
$ zip archive.zip directory/

# ✅ Includes all contents
$ zip -r archive.zip directory/
```

### Tip 2: Zip encryption is WEAK — don't rely on it
```bash
# zip uses ZipCrypto (easily crackable!)
# For real security, use:
$ 7z a -p -mhe=on secure.7z files/          # 7-Zip AES-256
$ tar -czvf - files/ | gpg -c > secure.tar.gz.gpg  # GPG
```

### Tip 3: Quick info about a zip
```bash
$ zipinfo archive.zip           # Detailed info
$ unzip -l archive.zip | tail -1  # Total file count
```

### Tip 4: Pipe from unzip
```bash
$ unzip -p archive.zip file.txt | grep "pattern"
$ unzip -p archive.zip data.csv | head -20
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Cross-platform (universal) | No Unix permission preservation |
| Individual file access | Weak encryption (ZipCrypto) |
| Built-in password protection | Less efficient than tar.gz |
| Split archive support | No incremental backup |
| Widely understood | Compression ratio lower than xz |

---

## 📍 Where & When to Use

| Scenario | Tool | Why |
|----------|------|-----|
| Share with Windows users | `zip` | Native support |
| Email attachment | `zip` | Universal |
| Linux system backup | `tar.gz` | Preserves permissions |
| Password-protect (weak) | `zip -e` | Quick protection |
| Password-protect (strong) | `7z` or `gpg` | AES-256 |
| CTF challenge | `fcrackzip` / `john` | Crack zip passwords |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `-r` for directories | Always `zip -r` for dirs |
| Trusting zip encryption | Use 7z/gpg for security |
| Extracting unknown zips | `unzip -l` first! (zip bomb check) |
| Password in command (`-P`) | Use `-e` (prompt) instead |
| Not preserving Linux permissions | Use tar.gz for Linux backups |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Create a zip of a directory with `zip -r`
2. List contents with `unzip -l`
3. Extract to a specific directory

### 🟡 Intermediate
4. Create a password-protected zip
5. Exclude patterns when zipping
6. Split a large zip into parts

### 🔴 Advanced
7. Crack a zip password with fcrackzip/john
8. Detect a zip bomb by checking expanded size
9. Convert between zip and tar.gz formats

---

## 🧠 Cheat Sheet

```
CREATE:
  zip -r archive.zip dir/         → Zip directory
  zip -r9 archive.zip dir/        → Best compression
  zip -re secure.zip dir/         → With password
  zip -r archive.zip dir/ -x "*.git*"  → With exclusions

EXTRACT:
  unzip archive.zip               → Extract here
  unzip archive.zip -d /dest/     → Extract to dir
  unzip archive.zip "file.txt"    → Extract single file
  unzip -o archive.zip            → Overwrite silently

LIST/TEST:
  unzip -l archive.zip            → List contents
  unzip -t archive.zip            → Test integrity

MODIFY:
  zip -u archive.zip newfile      → Update/add file
  zip -d archive.zip oldfile      → Delete from archive

CRACK:
  fcrackzip -u -D -p wordlist.txt encrypted.zip
  zip2john encrypted.zip > hash; john hash

SPLIT:
  zip -r -s 100m big.zip dir/    → Split into 100MB parts
```

---

> **Previous**: [`gzip/bzip2/xz` ←](./57_gzip_bzip2_xz.md) | **Next**: [`systemctl` →](../10_advanced/59_systemctl.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
