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
