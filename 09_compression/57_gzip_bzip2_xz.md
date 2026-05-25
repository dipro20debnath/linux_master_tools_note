# 🛠️ `gzip` / `bzip2` / `xz` — Compression Tools | Linux Master Note

> **Squeeze files to a fraction of their size. Three compression algorithms for three different needs — speed vs ratio vs balance.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--compression-algorithms)
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

### The Big Three:
| Tool | Algorithm | Speed | Compression | Extension |
|------|-----------|-------|-------------|-----------|
| `gzip` | DEFLATE (LZ77 + Huffman) | ⚡⚡ Fast | Good | `.gz` |
| `bzip2` | Burrows-Wheeler | ⚡ Moderate | Better | `.bz2` |
| `xz` | LZMA2 | 🐢 Slow | **Best** | `.xz` |

### Modern additions:
| Tool | Speed | Compression | Use Case |
|------|-------|-------------|----------|
| `zstd` | ⚡⚡⚡ Fastest | Great | Facebook's standard |
| `lz4` | ⚡⚡⚡⚡ Ultra-fast | OK | Real-time compression |
| `pigz` | ⚡⚡ Fast (multi-core) | Good | Parallel gzip |

### Key rule:
> These tools compress **single files only**. For directories, use `tar` first, then compress — or use `tar -czvf` which combines both.

---

## 📖 Theory — Compression Algorithms

### Benchmark (compressing 1GB text file):
| Tool | Compressed Size | Compress Time | Decompress Time |
|------|----------------|---------------|-----------------|
| gzip | 250MB | 15s | 5s |
| bzip2 | 200MB | 60s | 25s |
| xz | 150MB | 180s | 10s |
| zstd | 220MB | 5s | 3s |

### How compression works:
```
Original: AAABBBCCAAABBB (14 chars)
                ↓
Compressed: 3A3B2C3A3B (10 chars) — Run-Length Encoding

Real algorithms are MUCH more sophisticated:
- gzip: Sliding window, Huffman coding
- bzip2: Block sorting, Move-to-front, Huffman
- xz: Dictionary compression (LZMA2)
```

---

## 🧰 Syntax & Options

### `gzip`:
| Command | Description |
|---------|-------------|
| `gzip FILE` | Compress (replaces original!) |
| `gunzip FILE.gz` | Decompress |
| `gzip -d FILE.gz` | Decompress (same as gunzip) |
| `gzip -k FILE` | **Keep** original file |
| `gzip -r DIR` | Compress all files in directory |
| `gzip -1` to `-9` | Compression level (1=fast, 9=best) |
| `gzip -l FILE.gz` | List compression info |
| `gzip -c FILE` | Write to stdout (pipe-friendly) |
| `gzip -t FILE.gz` | Test integrity |
| `gzip -v FILE` | Verbose (show ratio) |
| `zcat FILE.gz` | View without decompressing |
| `zgrep PATTERN FILE.gz` | Grep inside compressed file |
| `zless FILE.gz` | Page through compressed file |

### `bzip2`:
| Command | Description |
|---------|-------------|
| `bzip2 FILE` | Compress |
| `bunzip2 FILE.bz2` | Decompress |
| `bzip2 -k FILE` | Keep original |
| `bzip2 -1` to `-9` | Compression level |
| `bzip2 -t FILE.bz2` | Test integrity |
| `bzcat FILE.bz2` | View without decompressing |
| `bzgrep PATTERN FILE.bz2` | Grep compressed |

### `xz`:
| Command | Description |
|---------|-------------|
| `xz FILE` | Compress |
| `unxz FILE.xz` | Decompress |
| `xz -k FILE` | Keep original |
| `xz -0` to `-9` | Compression level |
| `xz -e` | Extreme compression |
| `xz -T N` | Use N threads (multi-core!) |
| `xz -T 0` | Use ALL cores |
| `xz -l FILE.xz` | List info |
| `xz -t FILE.xz` | Test integrity |
| `xzcat FILE.xz` | View without decompressing |
| `xzgrep PATTERN FILE.xz` | Grep compressed |

---

## 🟢 Basic Usage

### Compress
```bash
# gzip (fast, default choice)
$ gzip file.txt                # → file.txt.gz (original DELETED!)
$ gzip -k file.txt             # → file.txt.gz (original KEPT)
$ gzip -9 file.txt             # Best compression

# bzip2 (better compression)
$ bzip2 file.txt               # → file.txt.bz2
$ bzip2 -k file.txt            # Keep original

# xz (best compression)
$ xz file.txt                  # → file.txt.xz
$ xz -k file.txt               # Keep original
$ xz -9e file.txt              # Maximum compression
```

### Decompress
```bash
# gzip
$ gunzip file.txt.gz           # → file.txt
$ gzip -d file.txt.gz          # Same thing

# bzip2
$ bunzip2 file.txt.bz2

# xz
$ unxz file.txt.xz
$ xz -d file.txt.xz            # Same thing
```

### View without decompressing
```bash
$ zcat file.txt.gz              # gzip
$ bzcat file.txt.bz2            # bzip2
$ xzcat file.txt.xz             # xz

# Search inside compressed
$ zgrep "error" /var/log/syslog.*.gz
$ zless /var/log/syslog.1.gz
```

---

## 🟡 Intermediate Usage

### Compression levels
```bash
# gzip: 1 (fastest) to 9 (best compression)
$ gzip -1 file.txt      # Fast, larger file
$ gzip -6 file.txt      # Default
$ gzip -9 file.txt      # Best, slower

# Compare:
$ for i in 1 3 6 9; do
    cp original.txt test.txt
    gzip -$i test.txt
    echo "Level $i: $(ls -lh test.txt.gz | awk '{print $5}')"
    gunzip test.txt.gz
done
```

### View compression stats
```bash
$ gzip -l file.txt.gz
  compressed uncompressed  ratio  uncompressed_name
       12345         50000  75.3%  file.txt

$ xz -l file.txt.xz
Strms  Blocks   Compressed Uncompressed  Ratio  Check   Filename
    1       1     10,234 B     50,000 B  0.205  CRC64   file.txt.xz
```

### Multi-core compression
```bash
# xz with all CPU cores (MUCH faster!)
$ xz -T 0 large_file.dat       # Use all cores

# pigz — parallel gzip
$ pigz file.txt                 # Uses all cores automatically
$ pigz -p 4 file.txt            # Use 4 cores
$ unpigz file.txt.gz            # Decompress

# pbzip2 — parallel bzip2
$ pbzip2 file.txt
$ pbzip2 -d file.txt.bz2
```

### Pipe usage
```bash
# Compress stdin to file
$ cat large_file.txt | gzip > compressed.gz

# Decompress and pipe
$ zcat data.gz | grep "pattern" | wc -l

# Compress database dump
$ mysqldump -u root database | gzip > db_backup.sql.gz

# Decompress and restore
$ zcat db_backup.sql.gz | mysql -u root database
```

### Compress directory recursively
```bash
# gzip all files in directory (NOT a single archive!)
$ gzip -r /var/log/old/
# Each file compressed individually

# For single archive, use tar:
$ tar -czvf logs.tar.gz /var/log/old/
```

---

## 🔴 Advanced Usage

### Log rotation compression
```bash
# Compress old log files (common in logrotate)
$ find /var/log -name "*.log" -mtime +7 -exec gzip {} \;

# View rotated compressed logs
$ zcat /var/log/syslog.2.gz | grep "error"
$ zgrep "failed" /var/log/auth.log.*.gz
```

### Benchmark compression tools
```bash
#!/bin/bash
# compress_benchmark.sh — Compare compression tools
FILE=$1
echo "=== Compression Benchmark: $FILE ==="
echo "Original: $(ls -lh $FILE | awk '{print $5}')"
echo ""

for tool in gzip bzip2 xz zstd; do
    cp "$FILE" "/tmp/bench_test"
    TIME_START=$(date +%s%N)
    $tool -k "/tmp/bench_test" 2>/dev/null
    TIME_END=$(date +%s%N)
    ELAPSED=$(( (TIME_END - TIME_START) / 1000000 ))
    
    EXT=$(ls /tmp/bench_test.* 2>/dev/null | head -1)
    SIZE=$(ls -lh "$EXT" 2>/dev/null | awk '{print $5}')
    
    echo "$tool: $SIZE (${ELAPSED}ms)"
    rm -f /tmp/bench_test*
done
```

### Security — Compressed File Analysis 🔒
```bash
# Check if file is actually what extension says (CTF!)
$ file suspicious.gz
suspicious.gz: gzip compressed data, was "secret.txt"

$ file mystery.dat
mystery.dat: XZ compressed data    # Renamed xz!

# Test integrity (detect corruption/tampering)
$ gzip -t backup.tar.gz && echo "OK" || echo "CORRUPTED!"
$ xz -t archive.tar.xz
$ bzip2 -t data.tar.bz2

# Decompress bomb detection (decompression attack!)
# A tiny file that expands to TERABYTES
$ gzip -l suspicious.gz
  compressed uncompressed  ratio
         100 999999999999  100%    # ⚠️ DECOMPRESSION BOMB!
# → Check ratio before decompressing!
```

### zstd (modern alternative)
```bash
# Compress (fast!)
$ zstd file.txt                    # → file.txt.zst
$ zstd -19 file.txt                # Max compression
$ zstd -T0 file.txt                # All cores

# Decompress
$ unzstd file.txt.zst
$ zstd -d file.txt.zst

# With tar
$ tar --zstd -cvf archive.tar.zst dir/
$ tar --zstd -xvf archive.tar.zst
```

---

## 💡 Real World Pro Tips

### Tip 1: Always use `-k` to keep originals
```bash
$ gzip -k file.txt     # Don't delete original!
```

### Tip 2: Use `zcat`/`zgrep` for compressed logs
```bash
$ zgrep "error" /var/log/*.gz     # Search all compressed logs
$ zless /var/log/syslog.1.gz      # Browse compressed log
```

### Tip 3: Multi-core compression saves time
```bash
$ pigz large_file.dat              # gzip with all cores
$ xz -T 0 large_file.dat          # xz with all cores
```

### Tip 4: Check before decompressing unknown files
```bash
$ file mystery.dat                 # What is this really?
$ gzip -l mystery.gz              # Check compression ratio
```

---

## ✅ Pros & Cons

| Tool | ✅ Pros | ❌ Cons |
|------|---------|---------|
| **gzip** | Fast, universal, zcat/zgrep tools | Lower compression ratio |
| **bzip2** | Better compression | Slow, high memory |
| **xz** | Best compression, multi-thread | Slowest compression |
| **zstd** | Fastest with great ratio | Not universal yet |

---

## 📍 Where & When to Use

| Scenario | Tool | Why |
|----------|------|-----|
| Daily backups | `gzip` | Fast enough, good ratio |
| Software distribution | `xz` | Best compression |
| Real-time/streaming | `zstd` or `lz4` | Speed matters |
| Log compression | `gzip` | logrotate default |
| Maximum compression | `xz -9e` | Smallest file |
| Multi-core available | `pigz` or `xz -T0` | Use all CPUs |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `-k` (original deleted!) | Always `gzip -k` |
| Using gzip on directories | Use `tar -czvf` instead |
| Not using multi-core | `pigz`, `pbzip2`, `xz -T0` |
| Trusting file extensions | Use `file` command to verify |
| Extracting unknown .gz blindly | Check `gzip -l` first |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Compress and decompress a file with each tool
2. View compression ratios with `gzip -l`
3. Use `zcat` to view a compressed file

### 🟡 Intermediate
4. Compare compression levels (1 vs 6 vs 9)
5. Use multi-core compression with `pigz`/`xz -T0`
6. Search inside compressed logs with `zgrep`

### 🔴 Advanced
7. Write a compression benchmark script
8. Detect a decompression bomb
9. Set up log rotation with compression

---

## 🧠 Cheat Sheet

```
═══ GZIP (fast, good) ═══
gzip file              → Compress (deletes original!)
gzip -k file           → Compress (keep original)
gunzip file.gz         → Decompress
zcat file.gz           → View without decompress
zgrep "pat" file.gz    → Search compressed
gzip -l file.gz        → Compression info
gzip -9 file           → Best compression

═══ BZIP2 (slower, better) ═══
bzip2 -k file          → Compress
bunzip2 file.bz2       → Decompress
bzcat file.bz2         → View

═══ XZ (slowest, best) ═══
xz -k file             → Compress
unxz file.xz           → Decompress
xzcat file.xz          → View
xz -T 0 file           → Use ALL CPU cores
xz -9e file            → Maximum compression

═══ ZSTD (modern, fast+good) ═══
zstd file              → Compress
unzstd file.zst        → Decompress

═══ MULTI-CORE ═══
pigz file              → Parallel gzip
pbzip2 file            → Parallel bzip2
xz -T 0 file           → Multi-threaded xz
```

---

> **Previous**: [`tar` ←](./56_tar.md) | **Next**: [`zip/unzip` →](./58_zip_unzip.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
