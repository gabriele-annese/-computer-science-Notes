---
tags:
  - linux
  - easy
  - fuff
  - "#virtualenv"
cssclasses:
  - page-grid
  - pen-red
---
## Recon

**Fast** Nmap scanning
```shell
sudo nmap -T4 -sV -F -Pn -oN nmap 10.10.11.23
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-29 17:17 EDT
Nmap scan report for PermX.htb (10.10.11.23)
Host is up (0.091s latency).
Not shown: 98 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.85 seconds

```

- Port 22 ssh: OpenSSH 8.9p1 Ubuntu 
- Port 80 http: WebSite

### Fuzzing Subdomains
i fuzzing the subdomains using #fuff 
```shell
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -
u http://permx.htb -H "Host: FUZZ.permx.htb" -mc 200 
```

- mc: To match the status code 200
i fond the subdomains **lms**.

![[Pasted image 20240731233000.png]]
i added the subdomain to my host file (/etc/hosts). Now we have a login form ;).
![[Pasted image 20240731234048.png]]



## Exploit
/documentatio version
Use [[Python Virtual Env]] to execute this exploit

exploit https://github.com/m3m0o/chamilo-lms-unauthenticated-big-upload-rce-poc?tab=readme-ov-file

## Temp Notes
possible apache version vulnerable
Apache/2.4.52 (Ubuntu) Server at permx.htb Port 80
![[Pasted image 20240731224540.png]]
