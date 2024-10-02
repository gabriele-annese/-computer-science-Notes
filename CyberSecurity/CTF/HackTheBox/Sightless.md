# CTF Report: [SIGHTLESS]
*Date of test*: [2024/09/30]  
*Author*: [Annese Gabriele]  
*Difficulty Level*: [Easy]  

---

## 1. Overview
**Machine Name**: [SIGHTLESS]  
**Operating System**: [Linux]  
**IP Address**: [10.10.11.32]  
**Objective**: Gain root/administrator access to the machine and retrieve the flags.

---

## 2. Reconnaissance

### 2.1. Initial Nmap Scan
```bash
nmap -F -sV -sC -oN defaul_port_scan.txt 10.10.11.32
Nmap scan report for sightless.htb (10.10.11.32)
Host is up (0.096s latency).
Not shown: 97 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (sightless.htb FTP Server) [::ffff:10.10.11.32]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c9:6e:3b:8f:c6:03:29:05:e5:a0:ca:00:90:c9:5c:52 (ECDSA)
|_  256 9b:de:3a:27:77:3b:1b:e1:19:5f:16:11:be:70:e0:56 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Sightless.htb
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.94SVN%I=7%D=9/30%Time=66FB0CEB%P=x86_64-pc-linux-gnu%r(G
SF:enericLines,A0,"220\x20ProFTPD\x20Server\x20\(sightless\.htb\x20FTP\x20
SF:Server\)\x20\[::ffff:10\.10\.11\.32\]\r\n500\x20Invalid\x20command:\x20
SF:try\x20being\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try\x
SF:20being\x20more\x20creative\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Sep 30 16:42:12 2024 -- 1 IP address (1 host up) scanned in 68.44 seconds
```

### 2.1. Explore WebSite
Looking the source code of the web site i found one subdomain and the email

![[Pasted image 20240930224711.png]]
Add the subdomain to the file hots
![[Pasted image 20240930224834.png]]
### 2.3. Enum the subdomain
when try to brute force the directory the gobuster tool return this error
```
Error: the server returns a status code that matches the provided options for non existing urls. http://sqlpad.sightless.htb/521f0d2f-b469-4820-834c-7a00c89edf74 => 200 (Length: 722). To continue please exclude the status code or the length

```

![[Pasted image 20240930230624.png]]
For go forward i exclude the `722 length`
```bash
sudo gobuster dir -u http://sqlpad.sightless.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --exclude-length 722 -o directory_list 
```

i dont found nothig of interesteing

## 3. Exploit
Search online for CVE of SQLPad i found a [RCE](https://github.com/0xRoqeeb/sqlpad-rce-exploit-CVE-2022-0944/blob/main/README.md) vulnerability with `api\test-connection`. Connect directly with root
![[Pasted image 20241001000131.png]]

### 4. Privilege Escalation
After run linepeas i notice the /etc/shadow are readable and writable 

![[Pasted image 20241002233956.png]]

Try to crack password using John. First copy and paste the passwd and shadow file and use the **unshadow** tool for prepare input file to pass a John
```bash
unshadow passwd.txt shadow.txt >> inputJohn
```

Crack passwd
```bash
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt inputJohn
```

![[Pasted image 20241003001456.png]]
Connect with ssh and michael user and find the first flag
![[Pasted image 20241003001710.png]]
Now it possible intercat with docker

## Loot

| User    | Pwd              |
| ------- | ---------------- |
| root    | blindside        |
| michael | insaneclownposse |


| User | flag                             |
| ---- | -------------------------------- |
| user | 501dc92dbd0ba17555f2ce191802ba82 |
| root |                                  |

