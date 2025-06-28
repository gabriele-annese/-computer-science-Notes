---

Armed with the domain user information, it is next important to gather AD group information to see what privileges members of a group may have and even find nested groups or issues with group membership that could lead to unintended rights.

---

## Domain Groups

A quick check shows that our target domain, `INLANEFREIGHT.LOCAL` has 72 groups.

```powershell-session
PS C:\htb> Get-DomainGroup -Properties Name

name
----
Administrators
Users
Guests
Print Operators
Backup Operators
Replicator
Remote Desktop Users
Network Configuration Operators
Performance Monitor Users
Performance Log Users
Distributed COM Users
IIS_IUSRS
Cryptographic Operators
Event Log Readers
Certificate Service DCOM Access
RDS Remote Access Servers
RDS Endpoint Servers
RDS Management Servers
Hyper-V Administrators
Access Control Assistance Operators
Remote Management Users
System Managed Accounts Group
Storage Replica Administrators
Domain Computers
Domain Controllers
Schema Admins
Enterprise Admins
Cert Publishers
Domain Admins
Domain Users
Domain Guests
Group Policy Creator Owners
RAS and IAS Servers
Server Operators
Account Operators
Pre-Windows 2000 Compatible Access
Incoming Forest Trust Builders
Windows Authorization Access Group
Terminal Server License Servers
Allowed RODC Password Replication Group
Denied RODC Password Replication Group
Read-only Domain Controllers
Enterprise Read-only Domain Controllers
Cloneable Domain Controllers
Protected Users
Key Admins
Enterprise Key Admins
DnsAdmins
DnsUpdateProxy
LAPS Admins
Security Operations
Organization Management
Recipient Management
View-Only Organization Management
Public Folder Management
UM Management
Help Desk
Records Management
Discovery Management
Server Management
Delegated Setup
Hygiene Management
Compliance Management
Security Reader
Security Administrator
Exchange Servers
Exchange Trusted Subsystem
Managed Availability Servers
Exchange Windows Permissions
ExchangeLegacyInterop
Exchange Install Domain Servers
Network Team
```

Let's grab a full listing of the group names. Many of these are built-in, standard AD groups. The presence of some group shows us that Microsoft Exchange is present in the environment. An Exchange installation adds several groups to AD, some of which such as `Exchange Trusted Subsystem` and `Exchange Windows Permissions` are considered [high-value targets](https://github.com/gdedrouas/Exchange-AD-Privesc) due to the permissions that membership in these groups grants a user or computer. Other groups such as `Protected Users`, `LAPS Admins`, `Help Desk`, and `Security Operations` should be noted down for review.

We can use `Get-DomainGroupMember` to examine group membership in any given group. Again, when using the `SharpView` function for this, we can pass the `-Help` flag to see all of the parameters that this function accepts.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainGroupMember -Help

Get_DomainGroupMember -Identity <String[]> -DistinguishedName <String[]> -SamAccountName <String[]> -Name <String[]> -MemberDistinguishedName <String[]> -MemberName <String[]> -Domain <String> -Recurse <Boolean> -RecurseUsingMatchingRule <Boolean> -LDAPFilter <String> -Filter <String> -SearchBase <String> -ADSPath <String> -Server <String> -DomainController <String> -SearchScope <SearchScope> -ResultPageSize <Int32> -ServerTimeLimit <Nullable`1> -SecurityMasks <Nullable`1> -Tombstone <Boolean> -Credential <NetworkCredential>
```

A quick examination of the `Help Desk` group shows us that there are two members.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainGroupMember -Identity 'Help Desk'

[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainGroupMember] Get-DomainGroupMember filter string: (&(objectCategory=group)(|(samAccountName=Help Desk)))
[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainObject] Extracted domain 'INLANEFREIGHT.LOCAL' from 'CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLA
NEFREIGHT,DC=LOCAL'
[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainObject] Get-DomainComputer filter string: (&(|(distinguishedname=CN=Harry Jones,OU=Network Ops,OU=IT,OU=Emplo
yees,DC=INLANEFREIGHT,DC=LOCAL)))
[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainObject] Extracted domain 'INLANEFREIGHT.LOCAL' from 'CN=Amber Smith,OU=Contractors,OU=Employees,DC=INLANEFREI
GHT,DC=LOCAL'
[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainObject] Get-DomainComputer filter string: (&(|(distinguishedname=CN=Amber Smith,OU=Contractors,OU=Employees,D
C=INLANEFREIGHT,DC=LOCAL)))
GroupDomain                    : INLANEFREIGHT,LOCAL
GroupName                      : Help Desk
GroupDistinguishedName         : CN=Help Desk,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
MemberDomain                   : INLANEFREIGHT.LOCAL
MemberName                     : harry.jones
MemberDistinguishedName        : CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
MemberObjectClass              : user
MemberSID                      : S-1-5-21-2974783224-3764228556-2640795941-2040

GroupDomain                    : INLANEFREIGHT,LOCAL
GroupName                      : Help Desk
GroupDistinguishedName         : CN=Help Desk,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
MemberDomain                   : INLANEFREIGHT.LOCAL
MemberName                     : amber.smith
MemberDistinguishedName        : CN=Amber Smith,OU=Contractors,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
MemberObjectClass              : user
MemberSID                      : S-1-5-21-2974783224-3764228556-2640795941-1859
```

---

## Protected Groups

Next, we can look for all AD groups with the `AdminCount` attribute set to 1, signifying that this is a protected group.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainGroup -AdminCount

[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainGroup] Searching for adminCount=1
[Get-DomainGroup] filter string: (&(objectCategory=group)(admincount=1))
objectsid                      : {S-1-5-32-544}
grouptype                      : CREATED_BY_SYSTEM, DOMAIN_LOCAL_SCOPE, SECURITY
samaccounttype                 : ALIAS_OBJECT
objectguid                     : 4f86f787-7173-4a34-a317-3f69e2263f0d
name                           : Administrators
distinguishedname              : CN=Administrators,CN=Builtin,DC=INLANEFREIGHT,DC=LOCAL
whencreated                    : 7/26/2020 8:13:52 PM
whenchanged                    : 8/23/2020 4:28:44 AM
samaccountname                 : Administrators
member                         : {CN=S-1-5-21-888139820-103978830-333442103-1602,CN=ForeignSecurityPrincipals,D
REIGHT,DC=LOCAL, CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL, CN=Enterprise Admins,CN=Users,DC=INLANEFR
LOCAL, CN=Administrator,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
cn                             : {Administrators}
objectclass                    : {top, group}
objectcategory                 : CN=Group,CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
usnchanged                     : 124889
description                    : Administrators have complete and unrestricted access to the computer/domain
instancetype                   : 4
usncreated                     : 8200
admincount                     : 1
iscriticalsystemobject         : True
systemflags                    : -1946157056
dscorepropagationdata          : {7/30/2020 3:52:30 AM, 7/30/2020 3:09:16 AM, 7/30/2020 3:09:16 AM, 7/28/2020 1
, 1/1/1601 12:00:00 AM}

objectsid                      : {S-1-5-32-550}
grouptype                      : CREATED_BY_SYSTEM, DOMAIN_LOCAL_SCOPE, SECURITY
samaccounttype                 : ALIAS_OBJECT
objectguid                     : ae974502-7850-44ab-9518-f909f9526daa
name                           : Print Operators
distinguishedname              : CN=Print Operators,CN=Builtin,DC=INLANEFREIGHT,DC=LOCAL
whencreated                    : 7/26/2020 8:13:52 PM
whenchanged                    : 7/30/2020 3:52:30 AM
samaccountname                 : Print Operators
cn                             : {Print Operators}
objectclass                    : {top, group}
iscriticalsystemobject         : True
usnchanged                     : 61476
instancetype                   : 4
usncreated                     : 8212
objectcategory                 : CN=Group,CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
admincount                     : 1
description                    : Members can administer printers installed on domain controllers
systemflags                    : -1946157056
dscorepropagationdata          : {7/30/2020 3:52:30 AM, 7/30/2020 3:09:16 AM, 7/30/2020 3:09:16 AM, 7/28/2020 1
, 1/1/1601 12:00:00 AM}

<...SNIP...>
```

Another important check is to look for any managed security groups. These groups have delegated non-administrators the right to add members to AD security groups and [distribution groups](https://docs.microsoft.com/en-us/exchange/recipients-in-exchange-online/manage-distribution-groups/manage-distribution-groups) and is set by modifying the `managedBy` attribute. This check looks to see if a group has a manager set and if the user can add users to the group. This could be useful for lateral movement by gaining us access to additional resources. First, let's take a look at the list of managed security groups.

```powershell-session
PS C:\htb> Find-ManagedSecurityGroups | select GroupName

GroupName
---------
Security Operations
Organization Management
Recipient Management
View-Only Organization Management
Public Folder Management
UM Management
Help Desk
Records Management
Discovery Management
Server Management
Delegated Setup
Hygiene Management
Compliance Management
Security Reader
Security Administrator
```

Next, let's look at the `Security Operations` group and see if the group has a manager set. We can see that the user `joe.evans` is set as the group manager.

```powershell-session
PS C:\htb> Get-DomainManagedSecurityGroup

GroupName                : Security Operations
GroupDistinguishedName   : CN=Security Operations,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
ManagerName              : joe.evans
ManagerDistinguishedName : CN=Joe Evans,OU=Security,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
ManagerType              : User
ManagerCanWrite          : UNKNOWN

GroupName                : Organization Management
GroupDistinguishedName   : CN=Organization Management,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
ManagerName              : Organization Management
ManagerDistinguishedName : CN=Organization Management,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
ManagerType              : Group
ManagerCanWrite          : UNKNOWN

GroupName                : Recipient Management
GroupDistinguishedName   : CN=Recipient Management,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
ManagerName              : Organization Management
ManagerDistinguishedName : CN=Organization Management,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
ManagerType              : Group
ManagerCanWrite          : UNKNOWN

GroupName                : View-Only Organization Management
GroupDistinguishedName   : CN=View-Only Organization Management,OU=Microsoft Exchange Security
                           Groups,DC=INLANEFREIGHT,DC=LOCAL
ManagerName              : Organization Management
ManagerDistinguishedName : CN=Organization Management,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
ManagerType              : Group
ManagerCanWrite          : UNKNOWN

<...SNIP...>
```

Enumerating the ACLs set on this group, we can see that this user has `GenericWrite` privileges meaning that this user can modify group membership (add or remove users). If we gain control of this user account, we can add this account or any other account that we control to the group and inherit any privileges that it has in the domain.

Enumerating AD Groups

```powershell-session
PS C:\htb> ConvertTo-SID joe.evans

S-1-5-21-2974783224-3764228556-2640795941-1238
```

Enumerating AD Groups

```powershell-session
PS C:\htb> $sid = ConvertTo-SID joe.evans
PS C:\htb> Get-DomainObjectAcl -Identity 'Security Operations' | ?{ $_.SecurityIdentifier -eq $sid}

ObjectDN              : CN=Security Operations,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
ObjectSID             : S-1-5-21-2974783224-3764228556-2640795941-2127
ActiveDirectoryRights : ListChildren, ReadProperty, GenericWrite
BinaryLength          : 36
AceQualifier          : AccessAllowed
IsCallback            : False
OpaqueLength          : 0
AccessMask            : 131132
SecurityIdentifier    : S-1-5-21-2974783224-3764228556-2640795941-1238
AceType               : AccessAllowed
AceFlags              : ContainerInherit
IsInherited           : False
InheritanceFlags      : ContainerInherit
PropagationFlags      : None
AuditFlags            : None
```

---

## Local Groups

It is also important to check local group membership. Is our current user local admin or part of local groups on any hosts? We can get a list of the local groups on a host using `Get-NetLocalGroup`.

```powershell-session
PS C:\htb> Get-NetLocalGroup -ComputerName WS01 | select GroupName

GroupName
---------
Access Control Assistance Operators
Administrators
Backup Operators
Certificate Service DCOM Access
Cryptographic Operators
Distributed COM Users
Event Log Readers
Guests
Hyper-V Administrators
IIS_IUSRS
Network Configuration Operators
Performance Log Users
Performance Monitor Users
Power Users
Print Operators
RDS Endpoint Servers
RDS Management Servers
RDS Remote Access Servers
Remote Desktop Users
Remote Management Users
Replicator
Storage Replica Administrators
System Managed Accounts Group
Users
```

We can also enumerate the local group members on any given host using the `Get-NetLocalGroupMember` function.

```powershell-session
PS C:\htb> .\SharpView.exe Get-NetLocalGroupMember -ComputerName WS01

ComputerName                   : WS01
GroupName                      : Administrators
MemberName                     : WS01\Administrator
SID                            : S-1-5-21-3098764391-2955872655-3533479253-500
IsGroup                        : False
IsDomain                       : false

ComputerName                   : WS01
GroupName                      : Administrators
MemberName                     : INLANEFREIGHT\
SID                            : S-1-5-21-2974783224-3764228556-2640795941-512
IsGroup                        : False
IsDomain                       : true

ComputerName                   : WS01
GroupName                      : Administrators
MemberName                     : INLANEFREIGHT\
SID                            : S-1-5-21-2974783224-3764228556-2640795941-2040
IsGroup                        : False
IsDomain                       : true

ComputerName                   : WS01
GroupName                      : Administrators
MemberName                     : INLANEFREIGHT\
SID                            : S-1-5-21-2974783224-3764228556-2640795941-513
IsGroup                        : False
IsDomain                       : true
```

We see one non-RID 500 user in the local administrators group and use the `Convert-SidToName` function to convert the SID and reveal the `harry.jones` user.

```powershell-session
PS C:\htb> Convert-SidToName S-1-5-21-2974783224-3764228556-2640795941-2040

INLANEFREIGHT\harry.jones
```

We use this same function to check all the hosts that a given user has local admin access, though this can be done much quicker with another `PowerView`/`SharpView` function that we will cover later in this module.

```powershell-session
PS C:\htb> $sid = Convert-NameToSid harry.jones
PS C:\htb> $computers = Get-DomainComputer -Properties dnshostname | select -ExpandProperty dnshostname
PS C:\htb> foreach ($line in $computers) {Get-NetLocalGroupMember -ComputerName $line | ? {$_.SID -eq $sid}}

ComputerName : WS01.INLANEFREIGHT.LOCAL
GroupName    : Administrators
MemberName   : INLANEFREIGHT\harry.jones
SID          : S-1-5-21-2974783224-3764228556-2640795941-2040
IsGroup      : False
IsDomain     : True
```

---

## Pulling Date User Added to Group

PowerView cannot pull the date when a user was added to a group, but since we are enumerating groups here, we wanted to include it. This information isn't too helpful for an attacker. Still, adding information that can aid in Incident Response will make your report stand out and hopefully lead to repeat business. Having this information, if you notice a strange user as part of a group, defenders can search for Event ID [4728](https://social.technet.microsoft.com/wiki/contents/articles/17049.active-directory-event-id-4728-4729-when-user-added-or-removed-from-security-enabled-global-group.aspx)/[4738](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4738) on that date to find out who added the user, or Event ID [4624](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624) since the date added to see if anyone has logged in.

The module we generally use to pull this information is called `Get-ADGroupMemberDate` and can be downloaded [here](https://raw.githubusercontent.com/proxb/PowerShell_Scripts/master/Get-ADGroupMemberDate.ps1). Load this module up the same way you would `PowerView`.

Then run `Get-ADGroupMemberDate -Group "Help Desk" -DomainController DC01.INLANEFREIGHT.LOCAL`, if there is a specific user you want to pull, we recommend running `Get-ADGroupMemberDate -Group "Help Desk" -DomainController DC01.INLANEFREIGHT.LOCAL | ? { ($_.Username -match 'harry.jones') -And ($_.State -NotMatch 'ABSENT')

---
