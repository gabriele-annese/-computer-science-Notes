## Access Control Lists (ACLs)

[Access Control List](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control-lists) (ACL) settings themselves are called Access Control Entries (ACEs). Each ACE refers back to a user, group, or process (security principal) and defines the principal's rights.

There are two types of ACLs.

|**ACL**|**Description**|
|---|---|
|`Discretionary Access Control List (DACL)`|This defines which security principals are granted or denied access to an object.|
|`System Access Control Lists (SACL)`|These allow administrators to log access attempts made to secured objects.|

ACL (mis)-configurations may allow for chained object-to-object control. We can visualize unrolled membership of target groups, so-called `derivative admins`, who can derive admin rights from exploiting an AD attack chain.

AD Attack chains may include the following components:

- "Unprivileged" users (shadow admins) having administrative access on member servers or workstations.
- Privileged users having a logon session on these workstations and member servers.
- Other forms of object-to-object control include force password change, add group member, change owner, write ACE, and full control.

Below is an example of just some of the ACLs that can be set on a user object.

![image](https://academy.hackthebox.com/storage/modules/68/dcarter_perms1.png)

---

## ACL Abuse

Why do we care about ACLs? ACL abuse is a powerful attack vector for us as penetration testers. These types of misconfigurations often go unnoticed in corporate environments because they can be difficult to monitor and control. An organization may be unaware of overly permissive ACL settings for years before (hopefully) we discover them. Below are some of the example Active Directory object security permissions (supported by `BloodHound` and abusable with `SharpView`/`PowerView`):

- ForceChangePassword abused with `Set-DomainUserPassword`
- Add Members abused with `Add-DomainGroupMember`
- GenericAll abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- GenericWrite abused with `Set-DomainObject`
- WriteOwner abused with `Set-DomainObjectOwner`
- WriteDACL abused with `Add-DomainObjectACL`
- AllExtendedRights abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`

---

## Enumerating ACLs with Built-In Cmdlets

We can use the built-in `Get-ADUser` cmdlet to enumerate ACLs. For example, we can look at the ACL for a single domain user `daniel.carter` with this command.

```powershell-session
PS C:\htb> (Get-ACL "AD:$((Get-ADUser daniel.carter).distinguishedname)").access  | ? {$_.IdentityReference -eq "INLANEFREIGHT\cliff.moore"}

ActiveDirectoryRights : ReadProperty, WriteProperty, GenericExecute
InheritanceType       : All
ObjectType            : 00000000-0000-0000-0000-000000000000
InheritedObjectType   : 00000000-0000-0000-0000-000000000000
ObjectFlags           : None
AccessControlType     : Allow
IdentityReference     : INLANEFREIGHT\cliff.moore
IsInherited           : False
InheritanceFlags      : ContainerInherit
PropagationFlags      : None

ActiveDirectoryRights : ExtendedRight
InheritanceType       : All
ObjectType            : ab721a53-1e2f-11d0-9819-00aa0040529b
InheritedObjectType   : 00000000-0000-0000-0000-000000000000
ObjectFlags           : ObjectAceTypePresent
AccessControlType     : Allow
IdentityReference     : INLANEFREIGHT\cliff.moore
IsInherited           : False
InheritanceFlags      : ContainerInherit
PropagationFlags      : None
```

We can drill down further on this user to find all users with `WriteProperty` or `GenericAll` rights over the target user.

```powershell-session
PS C:\htb> (Get-ACL "AD:$((Get-ADUser daniel.carter).distinguishedname)").access  | ? {$_.ActiveDirectoryRights -match "WriteProperty" -or $_.ActiveDirectoryRights -match "GenericAll"} | Select IdentityReference,ActiveDirectoryRights -Unique | ft -W

IdentityReference                                                                                            ActiveDirectoryRights
-----------------                                                                                            ---------------------
BUILTIN\Administrators                         CreateChild, DeleteChild, Self, WriteProperty, ExtendedRight, Delete, GenericRead,
                                                                                                             WriteDacl, WriteOwner
INLANEFREIGHT\Domain Admins                 CreateChild, DeleteChild, Self, WriteProperty, ExtendedRight, GenericRead, WriteDacl,
                                                                                                                        WriteOwner
INLANEFREIGHT\Enterprise Admins             CreateChild, DeleteChild, Self, WriteProperty, ExtendedRight, GenericRead, WriteDacl,
                                                                                                                        WriteOwner
INLANEFREIGHT\cliff.moore                                                              ReadProperty, WriteProperty, GenericExecute
NT AUTHORITY\SELF                                                                       ReadProperty, WriteProperty, ExtendedRight
BUILTIN\Terminal Server License Servers                                                                ReadProperty, WriteProperty
INLANEFREIGHT\Cert Publishers                                                                          ReadProperty, WriteProperty
INLANEFREIGHT\Organization Management                                                                                WriteProperty
INLANEFREIGHT\Exchange Servers                                                                                       WriteProperty
INLANEFREIGHT\Exchange Servers                     CreateChild, DeleteChild, ListChildren, ReadProperty, WriteProperty, ListObject
INLANEFREIGHT\Exchange Servers                                                     ReadProperty, WriteProperty, ListObject, Delete
INLANEFREIGHT\Exchange Trusted Subsystem           CreateChild, DeleteChild, ListChildren, ReadProperty, WriteProperty, ListObject
INLANEFREIGHT\Exchange Trusted Subsystem                                                                             WriteProperty
```

---

## Enumerating ACLs with PowerView and SharpView

We can use `PowerView`/`SharpView` to perform the previous command much quicker. For example, `Get-DomainObjectACL` can be used on a user to return similar data.

```powershell-session
PS C:\htb> Get-DomainObjectAcl -Identity harry.jones -Domain inlanefreight.local -ResolveGUIDs


AceQualifier           : AccessAllowed
ObjectDN               : CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : CreateChild, DeleteChild, ListChildren
ObjectAceType          : ms-Exch-Active-Sync-Devices
ObjectSID              : S-1-5-21-2974783224-3764228556-2640795941-2040
InheritanceFlags       : ContainerInherit
BinaryLength           : 72
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent, InheritedObjectAceTypePresent
IsCallback             : False
PropagationFlags       : InheritOnly
SecurityIdentifier     : S-1-5-21-2974783224-3764228556-2640795941-2615
AccessMask             : 7
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit, InheritOnly
InheritedObjectAceType : inetOrgPerson
OpaqueLength           : 0

AceQualifier           : AccessAllowed
ObjectDN               : CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : CreateChild, DeleteChild, ListChildren
ObjectAceType          : ms-Exch-Active-Sync-Devices
ObjectSID              : S-1-5-21-2974783224-3764228556-2640795941-2040
InheritanceFlags       : ContainerInherit
BinaryLength           : 72
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent, InheritedObjectAceTypePresent
IsCallback             : False
PropagationFlags       : InheritOnly
SecurityIdentifier     : S-1-5-21-2974783224-3764228556-2640795941-2615
AccessMask             : 7
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit, InheritOnly
InheritedObjectAceType : User
OpaqueLength           : 0

<...SNIP...>
```

We can seek out ACLs on specific users and filter out results using the various AD filters covered in the `Active Directory LDAP` module. We can use the `Find-InterestingDomainAcl` to search out objects in the domain with modification rights over non-built-in objects. This command, too, produces a large amount of data and can either be filtered on for information about specific objects or saved to be examined offline.

```powershell-session
PS C:\htb> Find-InterestingDomainAcl -Domain inlanefreight.local -ResolveGUIDs


ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : User-Change-Password
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-2974783224-3764228556-2640795941-2618
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : User-Force-Change-Password
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-2974783224-3764228556-2640795941-2618
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : CreateChild, DeleteChild, ListChildren
ObjectAceType           : ms-Exch-Active-Sync-Devices
AceFlags                : ContainerInherit, InheritOnly
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-2974783224-3764228556-2640795941-2615
IdentityReferenceName   : Exchange Servers
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Servers,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

<...SNIP...>
```

Aside from users and computers, we should also look at the ACLs set on file shares. This could provide us with information about which users can access a specific share or permissions are set too loosely on a specific share, which could lead to sensitive data disclosure or other attacks.

```powershell-session
PS C:\htb> Get-NetShare -ComputerName SQL01

Name             Type Remark        ComputerName
----             ---- ------        ------------
ADMIN$     2147483648 Remote Admin  SQL01
C$         2147483648 Default share SQL01
DB_backups          0               SQL01
IPC$       2147483651 Remote IPC    SQL01
```

  Enumerating Domain ACLs

```powershell-session
PS C:\htb> Get-PathAcl "\\SQL01\DB_backups"


Path              : \\SQL01\DB_backups
FileSystemRights  : Read
IdentityReference : Local System
IdentitySID       : S-1-5-18
AccessControlType : Allow

Path              : \\SQL01\DB_backups
FileSystemRights  : Read
IdentityReference : BUILTIN\Administrators
IdentitySID       : S-1-5-32-544
AccessControlType : Allow

Path              : \\SQL01\DB_backups
FileSystemRights  : Read
IdentityReference : BUILTIN\Users
IdentitySID       : S-1-5-32-545
AccessControlType : Allow

Path              : \\SQL01\DB_backups
FileSystemRights  : AppendData/AddSubdirectory
IdentityReference : BUILTIN\Users
IdentitySID       : S-1-5-32-545
AccessControlType : Allow

Path              : \\SQL01\DB_backups
FileSystemRights  : WriteData/AddFile
IdentityReference : BUILTIN\Users
IdentitySID       : S-1-5-32-545
AccessControlType : Allow

Path              : \\SQL01\DB_backups
FileSystemRights  : GenericAll
IdentityReference : Creator Owner
IdentitySID       : S-1-3-0
AccessControlType : Allow
```

Aside from ACLs of specific users and computers that may allow us to fully control or grant us other permissions, we should also check the ACL of the domain object. A common attack called [DCSync](https://adsecurity.org/?p=1729) requires a user to be delegated a combination of the following three rights:

- Replicating Directory Changes (DS-Replication-Get-Changes)
- Replicating Directory Changes All (DS-Replication-Get-Changes-All)
- Replicating Directory Changes In Filtered Set (DS-Replication-Get-Changes-In-Filtered-Set)

We can use the `Get-ObjectACL` function to search for all users that have these rights.

```powershell-session
PS C:\htb> Get-ObjectACL "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ObjectAceType -match 'Replication-Get')} | Select-Object SecurityIdentifier | Sort-Object -Property SecurityIdentifier -Unique

SecurityIdentifier
------------------
S-1-5-18
S-1-5-21-2974783224-3764228556-2640795941-1883
S-1-5-21-2974783224-3764228556-2640795941-2601
S-1-5-21-2974783224-3764228556-2640795941-2616
S-1-5-21-2974783224-3764228556-2640795941-498
S-1-5-21-2974783224-3764228556-2640795941-516
S-1-5-21-2974783224-3764228556-2640795941-519
S-1-5-32-544
S-1-5-9
```

Once we have the SIDs we can convert the SID back to the user to see which accounts have these rights and determine whether or not this is intended and/or if we can abuse these rights.

```powershell-session
PS C:\htb> convertfrom-sid S-1-5-21-2974783224-3764228556-2640795941-1883

INLANEFREIGHT\frederick.walton
```

This can be done quickly to enumerate all users with this right.

```powershell-session
PS C:\htb> $dcsync = Get-ObjectACL "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ObjectAceType -match 'Replication-Get')} | Select-Object -ExpandProperty SecurityIdentifier | Select -ExpandProperty value
PS C:\htb> Convert-SidToName $dcsync

INLANEFREIGHT\frederick.walton
INLANEFREIGHT\Enterprise Read-only Domain Controllers
INLANEFREIGHT\Domain Controllers
INLANEFREIGHT\Organization Management
INLANEFREIGHT\Exchange Trusted Subsystem
BUILTIN\Administrators
Enterprise Domain Controllers
INLANEFREIGHT\Enterprise Admins
Local System
```

---

## Leveraging ACLs

As seen in this section, various ACE entries can be set within AD. Administrators may set some on purpose to grant fine-grained privileges over an object or set of objects. In contrast, others may result from misconfigurations or installation of a service such as Exchange, which makes many changes ACLs within the domain by default.

We may compromise a user with `GenericWrite` over a user or group and can leverage this to force change a user's password or add our account to a specific group to further our access. Any modifications such as these should be carefully noted down and mentioned in the final report so the client can make sure changes are reverted if we cannot during the assessment period. Also, a "destructive" action, such as changing a user's password, should be used sparingly and coordinated with the client to avoid disruptions.

If we find a user, group, or computer with `WriteDacl` privileges over an object, we can leverage this in several ways. For example, if we can compromise a member of an Exchange-related group such as `Exchange Trusted Subsystem` we will likely have `WriteDacl` privileges over the domain object itself and be able to grant an account we control `Replicating Directory Changes` and `Replicating Directory Change` permissions to an account that we control and perform a DCSync attack to fully compromise the domain by mimicking a Domain Controller to retrieve user NTLM password hashes for any account we choose.

If we find ourselves with `GenericAll`/`GenericWrite` privileges over a target user, a less destructive attack would be to set a fake SPN on the account and perform a targeted `Kerberoasting` attack or modify the account's `userAccountControl` not to require Kerberos pre-authentication and perform a targeted `ASREPRoasting attack`. These examples require the account to be using a weak password that can be cracked offline using a tool such as `Hashcat` with minimal effort but are much less destructive than changing a user's password and have a higher likelihood of going unnoticed.

If you perform a destructive action such as changing a user's password and can compromise the domain, you can `DCSync`, obtain the account's password history, and use `Mimikatz` to reset the account to the previous password using `LSADUMP::ChangeNTLM` or `LSADUMP::SetNTLM`.

If we find ourselves with `GenericAll`/`GenericWrite` on a computer, we can perform a Kerberos Resource-based Constrained Delegation attack.

Sometimes we will find that a user or even the entire `Domain Users` group has been granted write permissions over a specific group policy object. If we find this type of misconfiguration, and the GPO is linked to one or more users or computers, we can use a tool such as [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) to modify the target GPO to perform actions such as provisioning additional privileges to a user (such as `SeDebugPrivilege` to be able to perform targeted credential theft, or `SeTakeOwnershipPrivilege` to gain control over a sensitive file or file share), add a user we control as a local admin to a target host, add a computer startup script, and more. As discussed above, these modifications should be performed carefully in coordination with the client and noted in the final report to minimize disruptions.

This is a summary of the many options we have for abusing ACLs. This topic will be covered more in-depth in later modules.