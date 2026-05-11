# 🛠️ `curl` — Transfer Data from/to Servers | Linux Master Note

> **The Swiss Army knife of HTTP. `curl` talks to any server using any protocol — APIs, downloads, uploads, authentication, and hacking.**

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [Theory](#-theory--http-basics)
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

### What is `curl`?
`curl` (**C**lient for **URL**s) transfers data to/from servers using **25+ protocols** including HTTP, HTTPS, FTP, SFTP, SCP, SMTP, and more.

### Supported protocols:
HTTP, HTTPS, FTP, FTPS, SCP, SFTP, TFTP, LDAP, DICT, TELNET, FILE, IMAP, POP3, SMTP, SMB, and more.

### Why it matters:
- **API testing** — Send GET/POST/PUT/DELETE requests
- **Web scraping** — Download pages and data
- **DevOps** — Health checks, deployments
- **Pentesting** — Test web vulnerabilities, send payloads
- **Scripting** — Automate HTTP interactions

---

## 📖 Theory — HTTP Basics

### HTTP Methods:
| Method | Purpose | Example |
|--------|---------|---------|
| `GET` | Retrieve data | Fetch a webpage |
| `POST` | Submit data | Login form, API create |
| `PUT` | Update data | API update |
| `DELETE` | Remove data | API delete |
| `PATCH` | Partial update | Update single field |
| `HEAD` | Headers only | Check if resource exists |
| `OPTIONS` | Supported methods | CORS preflight |

### HTTP Status Codes:
| Range | Category | Examples |
|-------|----------|---------|
| `1xx` | Informational | 100 Continue |
| `2xx` | Success | 200 OK, 201 Created |
| `3xx` | Redirect | 301 Moved, 302 Found |
| `4xx` | Client Error | 400 Bad Request, 403 Forbidden, 404 Not Found |
| `5xx` | Server Error | 500 Internal, 502 Bad Gateway, 503 Unavailable |

---

## 🧰 Syntax & Options

```bash
curl [OPTIONS] URL
```

| Flag | Description |
|------|-------------|
| `-o FILE` | Save output to file |
| `-O` | Save with remote filename |
| `-L` | Follow redirects |
| `-s` | Silent (no progress bar) |
| `-S` | Show errors even with `-s` |
| `-v` | Verbose (show headers) |
| `-I` / `--head` | Headers only (HEAD request) |
| `-X METHOD` | Specify HTTP method |
| `-d DATA` | POST data |
| `-H "Header"` | Custom header |
| `-u user:pass` | Authentication |
| `-b "cookies"` | Send cookies |
| `-c FILE` | Save cookies to file |
| `-k` | Ignore SSL certificate errors |
| `-A "Agent"` | Set User-Agent |
| `-x proxy:port` | Use proxy |
| `-w FORMAT` | Custom output format |
| `--data-urlencode` | URL-encode POST data |
| `-F "file=@path"` | Multipart form upload |
| `--connect-timeout N` | Connection timeout |
| `-m N` | Max time for entire operation |
| `-r RANGE` | Byte range (resume download) |

---

## 🟢 Basic Usage

```bash
# Fetch a webpage
$ curl https://example.com

# Save to file
$ curl -o page.html https://example.com
$ curl -O https://example.com/file.zip    # Keep remote name

# Follow redirects
$ curl -L https://bit.ly/shortlink

# Show response headers
$ curl -I https://example.com
HTTP/2 200
content-type: text/html
server: nginx

# Verbose (see full request/response)
$ curl -v https://example.com

# Silent mode
$ curl -s https://api.example.com/data

# Get just HTTP status code
$ curl -s -o /dev/null -w "%{http_code}" https://example.com
200
```

---

## 🟡 Intermediate Usage

### POST requests
```bash
# POST form data
$ curl -X POST -d "username=dipro&password=secret" https://example.com/login

# POST JSON
$ curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"Dipro","email":"dipro@example.com"}' \
  https://api.example.com/users

# POST with URL encoding
$ curl --data-urlencode "query=hello world" https://api.example.com/search
```

### API interactions
```bash
# GET with auth header
$ curl -H "Authorization: Bearer YOUR_TOKEN" https://api.example.com/me

# PUT (update)
$ curl -X PUT \
  -H "Content-Type: application/json" \
  -d '{"name":"Updated Name"}' \
  https://api.example.com/users/1

# DELETE
$ curl -X DELETE https://api.example.com/users/1

# Basic auth
$ curl -u admin:password https://api.example.com/admin
```

### File upload
```bash
# Upload file (multipart form)
$ curl -F "file=@/path/to/upload.pdf" https://example.com/upload

# Upload with extra fields
$ curl -F "file=@photo.jpg" -F "description=My photo" https://example.com/upload
```

### Custom headers
```bash
$ curl -H "Accept: application/json" \
  -H "X-API-Key: abc123" \
  -H "User-Agent: MyApp/1.0" \
  https://api.example.com/data
```

### Cookie handling
```bash
# Send cookies
$ curl -b "session=abc123; token=xyz" https://example.com/dashboard

# Save cookies from response
$ curl -c cookies.txt https://example.com/login -d "user=dipro&pass=secret"

# Use saved cookies
$ curl -b cookies.txt https://example.com/dashboard
```

### Download with progress
```bash
# Resume interrupted download
$ curl -C - -O https://example.com/large_file.iso

# Limit speed
$ curl --limit-rate 1M -O https://example.com/file.zip

# Multiple files
$ curl -O https://example.com/file1.zip -O https://example.com/file2.zip
```

---

## 🔴 Advanced Usage

### Web Security Testing 🔒
```bash
# Test for SQL injection
$ curl "https://target.com/page?id=1' OR '1'='1"

# Test for XSS
$ curl "https://target.com/search?q=<script>alert(1)</script>"

# Test for LFI (Local File Inclusion)
$ curl "https://target.com/page?file=../../../../etc/passwd"

# Test for SSRF
$ curl "https://target.com/fetch?url=http://169.254.169.254/latest/meta-data/"

# Check security headers
$ curl -sI https://target.com | grep -iE "x-frame|x-xss|content-security|strict-transport|x-content-type"

# Test CORS
$ curl -s -I -H "Origin: https://evil.com" https://target.com/api | grep -i access-control
```

### Proxy and tunneling
```bash
# Through HTTP proxy
$ curl -x http://proxy:8080 https://example.com

# Through SOCKS5 proxy (Tor)
$ curl --socks5 127.0.0.1:9050 https://check.torproject.org

# Through Burp Suite
$ curl -x http://127.0.0.1:8080 -k https://target.com
```

### Response timing analysis
```bash
$ curl -s -o /dev/null -w "\
  DNS Lookup:   %{time_namelookup}s\n\
  Connect:      %{time_connect}s\n\
  TLS:          %{time_appconnect}s\n\
  First Byte:   %{time_starttransfer}s\n\
  Total:        %{time_total}s\n\
  Status:       %{http_code}\n\
  Size:         %{size_download} bytes\n" \
  https://example.com
```

### Certificate inspection
```bash
# View SSL certificate details
$ curl -vI https://example.com 2>&1 | grep -A 5 "Server certificate"

# Check certificate expiry
$ curl -vI https://example.com 2>&1 | grep "expire date"
```

---

## 💡 Real World Pro Tips

### Tip 1: Pretty-print JSON
```bash
$ curl -s https://api.example.com/data | python3 -m json.tool
$ curl -s https://api.example.com/data | jq '.'
```

### Tip 2: Quick public IP check
```bash
$ curl -s ifconfig.me
$ curl -s ipinfo.io | jq '.'
```

### Tip 3: Health check in scripts
```bash
if [ $(curl -s -o /dev/null -w "%{http_code}" https://myapp.com/health) -eq 200 ]; then
    echo "App is healthy"
else
    echo "App is DOWN!"
fi
```

### Tip 4: Download entire website
```bash
# Use wget instead — curl isn't ideal for recursive downloads
$ wget -r -l 3 https://example.com
```

---

## ✅ Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| 25+ protocol support | Complex syntax for advanced use |
| Perfect for API testing | Not ideal for recursive downloads |
| Scriptable and pipeable | No built-in cookie jar management |
| SSL/TLS support | Verbose error messages |
| Proxy and SOCKS support | JSON needs external tools (jq) |

---

## 📍 Where & When to Use

| Scenario | Use `curl`? | Alternative |
|----------|-----------|-------------|
| API testing | ✅ Yes | Postman, httpie |
| Download file | ✅ Yes | `wget` (simpler) |
| Web scraping | ⚠️ Basic | `wget -r`, Python |
| Security testing | ✅ Yes | Burp Suite |
| Health checks | ✅ Yes | — |
| File upload | ✅ Yes | — |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Not following redirects | Add `-L` |
| Forgetting Content-Type for JSON | `-H "Content-Type: application/json"` |
| SSL errors on self-signed certs | `-k` (but not in production!) |
| Not URL-encoding special chars | Use `--data-urlencode` |
| Mixing `-d` and `-F` | Use one or the other |

---

## 📝 Practice Exercises

### 🟢 Beginner
1. Fetch a webpage with `curl`
2. Get just the HTTP status code
3. Download a file and save it

### 🟡 Intermediate
4. POST JSON data to an API
5. Use cookie-based authentication
6. Upload a file with `-F`

### 🔴 Advanced
7. Test a website's security headers
8. Measure response timing with `-w`
9. Use curl through a proxy for pentesting

---

## 🧠 Cheat Sheet

```
GET:     curl URL
POST:    curl -X POST -d "data" URL
JSON:    curl -X POST -H "Content-Type: application/json" -d '{}' URL
HEADERS: curl -I URL
AUTH:    curl -u user:pass URL / -H "Authorization: Bearer TOKEN"
UPLOAD:  curl -F "file=@path" URL
SAVE:    curl -o file URL / curl -O URL
FOLLOW:  curl -L URL
SILENT:  curl -s URL
STATUS:  curl -so /dev/null -w "%{http_code}" URL
COOKIE:  curl -b "k=v" URL / curl -c jar.txt URL
PROXY:   curl -x proxy:port URL
TIMING:  curl -so /dev/null -w "%{time_total}" URL
CERT:    curl -k URL (skip verify)
```

---

> **Previous**: [`netstat/ss` ←](./35_netstat_ss.md) | **Next**: [`wget` →](./37_wget.md)

*📅 Last Updated: May 2026 | 🐧 Linux Master Tools by [Dipro Debnath](https://github.com/dipro20debnath)*
