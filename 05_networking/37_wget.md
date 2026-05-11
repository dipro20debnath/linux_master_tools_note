# 🛠️ `wget` — Non-Interactive Network Downloader | Linux Master Note

> **The download king. `wget` downloads files, mirrors websites, and resumes broken transfers — all non-interactively, perfect for scripts and automation.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--wget-vs-curl)
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

### `wget` vs `curl`:
| Feature | `wget` | `curl` |
|---------|--------|--------|
| Best for | **Downloading files** | **API/HTTP interactions** |
| Recursive download | ✅ Built-in | ❌ No |
| Resume downloads | ✅ Automatic | Manual (`-C -`) |
| Protocols | HTTP, HTTPS, FTP | 25+ protocols |
| Background mode | ✅ `-b` flag | ❌ No |
| Mirror website | ✅ `--mirror` | ❌ No |
| Pipe output | ❌ Not ideal | ✅ Great |

> 🎯 **Rule of thumb**: `wget` to download, `curl` to interact.

---

## 🧰 Syntax & Options

```bash
wget [OPTIONS] URL(s)
```

| Flag | Description |
|------|-------------|
| `-O FILE` | Save with specific filename |
| `-P DIR` | Save to directory |
| `-c` | **Continue/resume** interrupted download |
| `-b` | Run in **background** |
| `-q` | Quiet (no output) |
| `-v` | Verbose |
| `-r` | **Recursive** download |
| `-l DEPTH` | Recursion depth (default: 5) |
| `-k` | Convert links for offline viewing |
| `-p` | Download all page elements (CSS, images) |
| `--mirror` | Mirror a website (`-r -N -l inf --no-remove-listing`) |
| `-np` | Don't go to parent directory |
| `-nd` | Don't create directory structure |
| `-N` | Timestamping (only download if newer) |
| `--limit-rate=RATE` | Limit download speed |
| `-U AGENT` | Set User-Agent |
| `--header="H: V"` | Custom header |
| `--http-user=USER` | HTTP auth username |
| `--http-password=PASS` | HTTP auth password |
| `-i FILE` | Read URLs from file |
| `--no-check-certificate` | Skip SSL verification |
| `-e robots=off` | Ignore robots.txt |
| `--reject=LIST` | Reject file types |
| `--accept=LIST` | Accept only file types |
| `-t N` | Retry N times (0 = infinite) |
| `-w SECS` | Wait between requests |
| `--random-wait` | Random wait (0.5× to 1.5× of `-w`) |
| `--spider` | Don't download, just check if exists |

---

## 🟢 Basic Usage

```bash
# Download a file
$ wget https://example.com/file.zip

# Save with custom name
$ wget -O my_file.zip https://example.com/file.zip

# Save to specific directory
$ wget -P ~/Downloads/ https://example.com/file.zip

# Download silently
$ wget -q https://example.com/file.zip

# Resume interrupted download
$ wget -c https://example.com/large_file.iso

# Download in background
$ wget -b https://example.com/huge_file.iso
# Check progress: tail -f wget-log
```

---

## 🟡 Intermediate Usage

### Multiple downloads
```bash
# Multiple URLs
$ wget https://example.com/file1.zip https://example.com/file2.zip

# From file list
$ cat urls.txt
https://example.com/file1.zip
https://example.com/file2.zip
https://example.com/file3.zip

$ wget -i urls.txt
```

### Speed limiting
```bash
# Limit to 500KB/s
$ wget --limit-rate=500k https://example.com/file.zip

# Limit to 2MB/s
$ wget --limit-rate=2m https://example.com/file.zip
```

### Authentication
```bash
# HTTP basic auth
$ wget --http-user=admin --http-password=secret https://example.com/private/file.zip

# With cookies
$ wget --load-cookies cookies.txt https://example.com/protected/
$ wget --save-cookies cookies.txt --post-data "user=dipro&pass=secret" https://example.com/login
```

### Custom headers
```bash
$ wget --header="Authorization: Bearer TOKEN123" https://api.example.com/data
$ wget -U "Mozilla/5.0 (Windows NT 10.0)" https://example.com
```

### Check URL without downloading
```bash
$ wget --spider https://example.com/file.zip
Spider mode enabled. Check if remote file exists.
HTTP/1.1 200 OK
Length: 1048576 (1.0M) [application/zip]
Remote file exists.
```

### Retry on failure
```bash
# Retry 10 times, wait 5 seconds between
$ wget -t 10 -w 5 https://unstable-server.com/file.zip

# Infinite retry (never give up!)
$ wget -t 0 -w 10 https://example.com/file.zip
```

---

## 🔴 Advanced Usage

### Mirror entire website
```bash
# Full mirror
$ wget --mirror --convert-links --adjust-extension --page-requisites \
  --no-parent https://example.com/
# Or shorthand:
$ wget -mkpnp https://example.com/

# Mirror with polite crawling
$ wget --mirror -w 2 --random-wait -e robots=off https://example.com/
```

### Recursive download (specific content)
```bash
# Download only PDFs from a site
$ wget -r -A "*.pdf" https://example.com/documents/

# Download only images
$ wget -r -A "*.jpg,*.png,*.gif" https://example.com/gallery/

# Reject certain types
$ wget -r --reject="*.exe,*.msi" https://example.com/

# Limit depth
$ wget -r -l 2 https://example.com/     # Only 2 levels deep
```

### Pentesting — Website Cloning 🎯
```bash
# Clone a website for phishing analysis/testing
$ wget --mirror --convert-links --adjust-extension \
  --page-requisites --no-parent -e robots=off \
  -U "Mozilla/5.0" https://target.com/

# Download robots.txt (recon!)
$ wget -q https://target.com/robots.txt -O -
Disallow: /admin/
Disallow: /api/internal/
Disallow: /backup/
# → These paths are interesting targets!

# Download wordlists for brute forcing
$ wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10k-most-common.txt
```

### FTP downloads
```bash
# Download from FTP
$ wget ftp://ftp.example.com/pub/file.zip

# With credentials
$ wget --ftp-user=admin --ftp-password=secret ftp://ftp.example.com/private/

# Mirror FTP directory
$ wget -r ftp://ftp.example.com/pub/
```

### Batch download script
```bash
#!/bin/bash
# download_batch.sh — Download with retry and logging
URLS="urls.txt"
LOG="download.log"
MAX_RETRY=5

while IFS= read -r url; do
    echo "$(date): Downloading $url" >> "$LOG"
    wget -c -t "$MAX_RETRY" -q "$url" >> "$LOG" 2>&1
    if [ $? -eq 0 ]; then
        echo "$(date): SUCCESS - $url" >> "$LOG"
    else
        echo "$(date): FAILED - $url" >> "$LOG"
    fi
done < "$URLS"
```

---

## 💡 Real World Pro Tips

### Tip 1: Always use `-c` for large files
```bash
# If download breaks, just run again — it resumes!
$ wget -c https://example.com/4GB_file.iso
```

### Tip 2: Polite crawling (don't get banned)
```bash
$ wget -r -w 2 --random-wait -e robots=off -U "Mozilla/5.0" https://example.com
```

### Tip 3: Download to stdout (pipe)
```bash
$ wget -qO- https://api.example.com/data | jq '.'
$ wget -qO- https://example.com/script.sh | bash    # ⚠️ Dangerous!
```

### Tip 4: Archive a website before it goes down
```bash
$ wget --mirror --warc-file=site_archive https://dying-website.com/
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Recursive/mirror downloads | HTTP only (no API features) |
| Auto-resume (`-c`) | No JSON/POST support (use curl) |
| Background mode | Slower than curl for single files |
| Non-interactive (great for scripts) | Website mirroring can be slow |
| Polite crawling options | Can overwhelm servers |

---

## 📍 Where & When to Use

| Scenario | Use `wget`? | Alternative |
|----------|-----------|-------------|
| Download file | ✅ Yes | `curl -O` |
| Mirror website | ✅ Yes | `httrack` |
| API calls | ❌ No | `curl` |
| Resume download | ✅ Yes | `curl -C -` |
| Batch downloads | ✅ Yes | `aria2c` (faster) |
| Download wordlists | ✅ Yes | — |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `-c` for large files | Always use `-c` for resume |
| Aggressive crawling (get banned) | Add `-w 2 --random-wait` |
| Not setting User-Agent | Use `-U "Mozilla/5.0..."` |
| Downloading to wrong directory | Use `-P /target/dir/` |
| Trusting piped scripts | Never `wget \| bash` without reading! |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Download a file from the internet
2. Resume an interrupted download with `-c`
3. Download silently in background with `-bq`

### 🟡 Intermediate
4. Download only PDF files from a website recursively
5. Mirror a small website with `--mirror`
6. Download files from a URL list

### 🔴 Advanced
7. Clone a website with all assets for offline viewing
8. Write a batch download script with retry and logging
9. Download and analyze a target's `robots.txt`

---

## 🧠 Cheat Sheet

```
DOWNLOAD:
  wget URL                    → Download file
  wget -O name URL            → Custom filename
  wget -P dir/ URL            → Save to directory
  wget -c URL                 → Resume download
  wget -b URL                 → Background
  wget -q URL                 → Silent
  wget -i urls.txt            → From file list

RECURSIVE:
  wget -r URL                 → Recursive download
  wget --mirror URL           → Full mirror
  wget -r -A "*.pdf" URL      → Only PDFs
  wget -r -l 2 URL            → Depth limit 2

MIRROR WEBSITE:
  wget --mirror -k -p -np URL → Offline-ready clone

SPEED/POLITENESS:
  wget --limit-rate=500k URL  → Speed limit
  wget -w 2 --random-wait URL → Polite crawling
  wget -t 0 URL               → Infinite retry

AUTH:
  wget --http-user=u --http-password=p URL
  wget --header="Authorization: Bearer TOKEN" URL
```

---

> **Previous**: [`curl` ←](./36_curl.md) | **Next**: [`ssh` →](./38_ssh.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
