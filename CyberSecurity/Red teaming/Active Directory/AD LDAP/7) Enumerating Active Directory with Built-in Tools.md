Proper enumeration is key for all penetration testing and red teaming assessments. Enumerating AD, especially large corporate environments with many hosts, users, and services, can be quite a daunting task and provide an overwhelming amount of data. Several built-in Windows tools can be used by sysadmins and pentesters to enumerate AD. Open source tools have been created based on the same enumeration techniques. Many of these tools (such as SharpView, BloodHound, and, PingCastle) can be utilized to expedite the enumeration process and accurately present the data in a consumable and actionable format. Knowledge of multiple tools and "offense in-depth" is important if you must live off the land on an assessment or detections are in place for certain tools.

---

## User-Account-Control (UAC) Attributes

User-Account-Control Attributes control the behavior of domain accounts. These values are not to be confused with the Windows User Account Control technology. Many of these UAC attributes have security relevance:

![image](https://academy.hackthebox.com/storage/modules/22/UAC1.png)

We can enumerate these values with built-in AD cmdlets:

#### PowerShell - Built-in AD Cmdlets

  Enumerating Active Directory with Built-in Tools

```powershell-session
PS C:\htb> Get-ADUser -Filter {adminCount -gt 0} -Properties admincount,useraccountcontrol | select Name,useraccountcontrol

Name           useraccountcontrol
----           ------------------
Administrator               66048
krbtgt                      66050
daniel.carter                 512
sqlqa                         512
svc-backup                  66048
svc-secops                  66048
cliff.moore                 66048
svc-ata                       512
svc-sccm                      512
mrb3n                         512
sarah.lafferty                512
Jenna Smith               4260384
Harry Jones                 66080
pixis                         512
Cry0l1t3                      512
knightmare                    512
```

We still need to convert the `useraccountcontrol` values into their corresponding flags to interpret them. This can be done with this [script](https://academy.hackthebox.com/storage/resources/Convert-UserAccountControlValues.zip). Let's take the user `Jenna Smith` with `useraccountcontrol` value `4260384` as an example.

#### PowerShell - UAC Values

  Enumerating Active Directory with Built-in Tools

```powershell-session
PS C:\htb> .\Convert-UserAccountControlValues.ps1

Please provide the userAccountControl value: : 4260384

Name                           Value
----                           -----
PASSWD_NOTREQD                 32
NORMAL_ACCOUNT                 512
DONT_EXPIRE_PASSWORD           65536
DONT_REQ_PREAUTH               4194304
```

We can also use [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1) (which will be covered in-depth in subsequent modules) to enumerate these values. We can see that some of the users match the default value of `512` or `Normal_Account` while others would need to be converted. The value for `jenna.smith` does match what our conversion script provided.

`PowerView` can be found in the `c:\tools` directory on the target host. To load the tool, open a PowerShell console, navigate to the tools directory, and import `PowerView` using the command `Import-Module .\PowerView.ps1`.

#### PowerView - Domain Accounts

  Enumerating Active Directory with Built-in Tools

```powershell-session
PS C:\htb> Get-DomainUser * -AdminCount | select samaccountname,useraccountcontrol

samaccountname                                                     useraccountcontrol
--------------                                                     ------------------
Administrator                                    NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
krbtgt                           ACCOUNTDISABLE, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
daniel.carter                                                          NORMAL_ACCOUNT
sqlqa                                                                  NORMAL_ACCOUNT
svc-backup                                       NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
svc-secops                                       NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
cliff.moore                                      NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
svc-ata                                                                NORMAL_ACCOUNT
svc-sccm                                                               NORMAL_ACCOUNT
mrb3n                                                                  NORMAL_ACCOUNT
sarah.lafferty                                                         NORMAL_ACCOUNT
jenna.smith    PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH
harry.jones                      PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
pixis                                                                  NORMAL_ACCOUNT
Cry0l1t3                                                               NORMAL_ACCOUNT
knightmare                                                             NORMAL_ACCOUNT
```

---

## Enumeration Using Built-In Tools

Tools that sysadmins are themselves likely to use, such as the PowerShell AD Module, the Sysinternals Suite, and AD DS Tools, are likely to be whitelisted and fly under the radar, especially in more mature environments. Several built-in tools can be leveraged for AD enumeration, including:

`DS Tools` is available by default on all modern Windows operating systems but required domain connectivity to perform enumeration activities.

#### DS Tools

  Enumerating Active Directory with Built-in Tools

```cmd-session
C:\htb> dsquery user "OU=Employees,DC=inlanefreight,DC=local" -name * -scope subtree -limit 0 | dsget user -samid -
pwdneverexpires | findstr /V no

  samid                  pwdneverexpires
  svc-backup             yes
  svc-scan               yes
  svc-secops             yes
  sql-test               yes
  cliff.moore            yes
  margaret.harris        yes
  
  <SNIP>
  
dsget succeeded
```

The `PowerShell Active Directory module` is a group of cmdlets used to manage Active Directory. The installation of the AD PowerShell module requires administrative access.

#### AD PowerShell Module

  Enumerating Active Directory with Built-in Tools

```powershell-session
PS C:\htb> Get-ADUser -Filter * -SearchBase 'OU=Admin,DC=inlanefreight,dc=local'

DistinguishedName : CN=wilford.stewart,OU=Admin,DC=INLANEFREIGHT,DC=LOCAL
Enabled           : True
GivenName         :
Name              : wilford.stewart
ObjectClass       : user
ObjectGUID        : 1f54c02c-2fb4-48b6-a89c-38b6b0c54147
SamAccountName    : wilford.stewart
SID               : S-1-5-21-2974783224-3764228556-2640795941-2121
Surname           :
UserPrincipalName :

DistinguishedName : CN=trisha.duran,OU=Admin,DC=INLANEFREIGHT,DC=LOCAL
Enabled           : True
GivenName         :
Name              : trisha.duran
ObjectClass       : user
ObjectGUID        : 7a8db2bb-7b24-4f79-a3fe-7b49408bc7bf
SamAccountName    : trisha.duran
SID               : S-1-5-21-2974783224-3764228556-2640795941-2122
Surname           :
UserPrincipalName :

<SNIP>
```

`Windows Management Instrumentation` (WMI) can also be used to access and query objects in Active Directory. Many scripting languages can interact with the WMI AD provider, but PowerShell makes this very easy.

#### Windows Management Instrumentation (WMI)

  Enumerating Active Directory with Built-in Tools

```powershell-session
PS C:\htb> Get-WmiObject -Class win32_group -Filter "Domain='INLANEFREIGHT'" | Select Caption,Name

Caption                                               Name
-------                                               ----
INLANEFREIGHT\Cert Publishers                         Cert Publishers
INLANEFREIGHT\RAS and IAS Servers                     RAS and IAS Servers
INLANEFREIGHT\Allowed RODC Password Replication Group Allowed RODC Password Replication Group
INLANEFREIGHT\Denied RODC Password Replication Group  Denied RODC Password Replication Group
INLANEFREIGHT\DnsAdmins                               DnsAdmins
INLANEFREIGHT\$6I2000-MBUUOKUK1E1O                    $6I2000-MBUUOKUK1E1O
INLANEFREIGHT\Cloneable Domain Controllers            Cloneable Domain Controllers
INLANEFREIGHT\Compliance Management                   Compliance Management
INLANEFREIGHT\Delegated Setup                         Delegated Setup
INLANEFREIGHT\Discovery Management                    Discovery Management
INLANEFREIGHT\DnsUpdateProxy                          DnsUpdateProxy
INLANEFREIGHT\Domain Admins                           Domain Admins
INLANEFREIGHT\Domain Computers                        Domain Computers
INLANEFREIGHT\Domain Controllers                      Domain Controllers
INLANEFREIGHT\Domain Guests                           Domain Guests
INLANEFREIGHT\Domain Users                            Domain Users
INLANEFREIGHT\Enterprise Admins                       Enterprise Admins
INLANEFREIGHT\Enterprise Key Admins                   Enterprise Key Admins
INLANEFREIGHT\Enterprise Read-only Domain Controllers Enterprise Read-only Domain Controllers
INLANEFREIGHT\Exchange Servers                        Exchange Servers
INLANEFREIGHT\Exchange Trusted Subsystem              Exchange Trusted Subsystem
INLANEFREIGHT\Exchange Windows Permissions            Exchange Windows Permissions
INLANEFREIGHT\ExchangeLegacyInterop                   ExchangeLegacyInterop
INLANEFREIGHT\Group Policy Creator Owners             Group Policy Creator Owners
INLANEFREIGHT\Help Desk                               Help Desk
INLANEFREIGHT\Hygiene Management                      Hygiene Management
INLANEFREIGHT\Key Admins                              Key Admins
INLANEFREIGHT\LAPS Admins                             LAPS Admins
INLANEFREIGHT\Managed Availability Servers            Managed Availability Servers
INLANEFREIGHT\Organization Management                 Organization Management
INLANEFREIGHT\Protected Users                         Protected Users

<SNIP>
```

`Active Directory Service Interfaces` (ADSI) is a set of COM interfaces that can query Active Directory. PowerShell again provides an easy way to interact with it.

#### AD Service Interfaces (ADSI)

  Enumerating Active Directory with Built-in Tools

```powershell-session
PS C:\htb> ([adsisearcher]"(&(objectClass=Computer))").FindAll() | select Path

Path
----
LDAP://CN=DC01,OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL
LDAP://CN=EXCHG01,OU=Mail Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL
LDAP://CN=SQL01,OU=SQL Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL
LDAP://CN=WS01,OU=Staff Workstations,OU=Workstations,DC=INLANEFREIGHT,DC=LOCAL
LDAP://CN=DC02,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL
```
	