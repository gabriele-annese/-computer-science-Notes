# Enumeration

## Nmap
Scan open ports
```bash
nmap -sC -sV -A -oA nmap/scan.txt 10.129.211.144

Nmap scan report for 10.129.211.144
Host is up (0.0081s latency).                
Not shown: 989 closed tcp ports (reset)                 
PORT     STATE SERVICE      VERSION            
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-05-25 20:24:35Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped     
```
the domain is `htb.local` 
## Ladapsearch Users Enum
Check if the LDAP service allow anonymous authentication
```bash
ldapsearch -H LDAP://10.129.211.144 -x -s base namingcontexts
```

![[Pasted image 20250525232229.png]]

The `namingcontexts` attribute lists the **base Distinguished Name (DNs)** of all **naming contexts** that the LDAP server knows about.

Think of naming context as **entry point or "root containers"** in the LDAP directory tree where different categories of data are stored. Each one represents different **subtree** of the directory

So in now we can query the context of `htb` and  `local` **root containers**.

### Create a users list with ldapsearch
We can retrieve a users list with this query and save that in a file
```bash
ldapsearch -H LDAP://10.129.211.144 -x -b 'DC=htb,DC=local' '(objectClass=User)' sAMAccountName | grep sAMAccountName | awk '{print $2}' > users_list
```

- `-x`: this flags enable anonymous authentication
- `-b`: this flags specified the "root containers" where preform the LDAP query.

`awk '{print $2}'` means take only the second column of the output.

Clear the list of the useless name like name machine default users etc..

Clear user list
![[Pasted image 20250525234215.png]]


