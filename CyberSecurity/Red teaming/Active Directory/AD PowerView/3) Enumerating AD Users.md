

When starting enumeration in an AD environment, arguably, the most important objects are domain users. Users have access to computers and are assigned permissions to perform a variety of functions throughout the domain. We need to control user accounts to move laterally and vertically within a network to reach the assessment goal.

---

## Key AD User Data Points

We can use `PowerView` and `SharpView` to enumerate a wealth of information about AD users. We can start by getting a count of how many users are in the target domain. We switch between the two tools frequently in this course because it is important to know them both. Sometimes you won't be able to run powershell commands and at other times you may not be able to run executable's. Any time you see SharpView usage, it should be possible to do it in PowerView by just removing SharpView.exe. SharpView does not have the latest PowerSploit features, so it may not be possible to run PowerView commands within SharpView.


```powershell-session
PS C:\htb> (Get-DomainUser).count

1038
```

Next, let's explore the `Get-DomainUser` function. If we provide the `-Help` flag to any `SharpView function, we can see all of the parameters that the function accepts.


```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -Help

Get_DomainUser -Identity <String[]> -DistinguishedName <String[]> -SamAccountName <String[]> -Name <String[]> -MemberDistinguishedName <String[]> -MemberName <String[]> -SPN <Boolean> -AdminCount <Boolean> -AllowDelegation <Boolean> -DisalowDelegation <Boolean> -TrustedToAuth <Boolean> -PreauthNotRequired <Boolean> -KerberosPreauthNotRequired <Boolean> -Noreauth <Boolean> -Domain <String> -LDAPFilter <String> -Filter <String> -Properties <String[]> -SearchBase <String> -ADPath <String> -Server <String> -DomainController <String> -SearchScope <SearchScope> -ResultPageSize <Int32> -ServerTimLimit <Nullable`1> -SecurityMasks <Nullable`1> -Tombstone <Boolean> -FindOne <Boolean> -ReturnOne <Boolean> -Credential <NetworkCredential> -Raw <Boolean> -UACFilter <UACEnum>
```

Below are some of the most important properties to gather about domain users. Let's take a look at the `harry.jones` user.

```powershell-session
PS C:\htb> Get-DomainUser -Identity harry.jones -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,mail,useraccountcontrol


name                 : Harry Jones
samaccountname       : harry.jones
description          :
memberof             : {CN=Network Team,CN=Users,DC=INLANEFREIGHT,DC=LOCAL, CN=Help Desk,OU=Microsoft Exchange
                       Security Groups,DC=INLANEFREIGHT,DC=LOCAL, CN=Security
                       Operations,CN=Users,DC=INLANEFREIGHT,DC=LOCAL, CN=LAPS
                       Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL...}
whencreated          : 7/27/2020 7:35:59 PM
pwdlastset           : 7/30/2020 2:33:04 PM
lastlogontimestamp   : 8/9/2020 10:52:42 PM
accountexpires       : 12/31/1600 7:00:00 PM
admincount           : 1
userprincipalname    : harry.jones@inlanefreight
serviceprincipalname :
mail                 :
useraccountcontrol   : PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
```

It is useful to enumerate these properties for ALL domain users and export them to a CSV file for offline processing.

```powershell-session
PS C:\htb> Get-DomainUser * -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,mail,useraccountcontrol | Export-Csv .\inlanefreight_users.csv -NoTypeInformation
```

Once we have gathered information on all users, we can begin to perform more specific user enumeration by obtaining a list of users that do not require Kerberos pre-authentication and can be subjected to an ASREPRoast attack.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -KerberosPreauthNotRequired -Properties samaccountname,useraccountcontrol,memberof

[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainUser] Searching for user accounts that do not require kerberos preauthenticate
[Get-DomainUser] filter string: (&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304))
useraccountcontrol             : PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH
samaccountname                 : amber.smith
memberof                       : {CN=Help Desk,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL}

useraccountcontrol             : PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH
samaccountname                 : jenna.smith
memberof                       : {CN=Schema Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
```

Let's also gather information about users with Kerberos constrained delegation.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -TrustedToAuth -Properties samaccountname,useraccountcontrol,memberof

[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainUser] Searching for users that are trusted to authenticate for other principals
[Get-DomainUser] filter string: (&(samAccountType=805306368)(msds-allowedtodelegateto=*))
useraccountcontrol             : NORMAL_ACCOUNT
samaccountname                 : sqlprod
memberof                       : {CN=Protected Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}

useraccountcontrol             : PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
samaccountname                 : adam.jones
```

While we're at it, we can look for users that allow unconstrained delegation.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -LDAPFilter "(userAccountControl:1.2.840.113556.1.4.803:=524288)"

[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainUser] Using additional LDAP filter: (userAccountControl:1.2.840.113556.1.4.803:=524288)
[Get-DomainUser] filter string: (&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=524288))
objectsid                      : {S-1-5-21-2974783224-3764228556-2640795941-1110}
samaccounttype                 : USER_OBJECT
objectguid                     : f71224a5-baa7-4aec-bfe9-56778184dc63
useraccountcontrol             : NORMAL_ACCOUNT, TRUSTED_FOR_DELEGATION
accountexpires                 : 12/31/1600 7:00:00 PM
lastlogon                      : 12/31/1600 7:00:00 PM
pwdlastset                     : 7/27/2020 2:46:20 PM
lastlogoff                     : 12/31/1600 7:00:00 PM
badPasswordTime                : 12/31/1600 7:00:00 PM
name                           : sqldev
distinguishedname              : CN=sqldev,OU=Service Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
whencreated                    : 7/27/2020 6:46:20 PM
whenchanged                    : 8/14/2020 1:30:37 PM
samaccountname                 : sqldev
memberof                       : {CN=Protected Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
cn                             : {sqldev}
objectclass                    : {top, person, organizationalPerson, user}
ServicePrincipalName           : CIFS/roguecomputer.inlanefreight.local
logoncount                     : 0
codepage                       : 0
objectcategory                 : CN=Person,CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
dscorepropagationdata          : {7/30/2020 3:09:16 AM, 7/30/2020 3:09:16 AM, 7/28/2020 1:45:00 AM, 7/28/2020 1:34:13 AM
, 7/14/1601 10:36:49 PM}
usnchanged                     : 110783
instancetype                   : 4
badpwdcount                    : 0
usncreated                     : 14648
countrycode                    : 0
primarygroupid                 : 513
```

We can also check for any domain users with sensitive data such as a password stored in the description field.

Enumerating AD Users

```powershell-session
PS C:\htb> Get-DomainUser -Properties samaccountname,description | Where {$_.description -ne $null}

samaccountname description
-------------- -----------
Administrator  Built-in account for administering the computer/domain
Guest          Built-in account for guest access to the computer/domain
DefaultAccount A user account managed by the system.
krbtgt         Key Distribution Center Service Account
svc-sccm       **Do not change password** 03/04/2015 N3ssu$_svc2014!
```

Next, let's enumerate any users with Service Principal Names (SPNs) that could be subjected to a Kerberoasting attack.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -SPN -Properties samaccountname,memberof,serviceprincipalname

[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainUser] Searching for non-null service principal names
[Get-DomainUser] filter string: (&(samAccountType=805306368)(servicePrincipalName=*))
samaccountname                 : sqldev
memberof                       : {CN=Protected Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
ServicePrincipalName           : CIFS/roguecomputer.inlanefreight.local

samaccountname                 : adam.jones
ServicePrincipalName           : IIS_dev/inlanefreight.local:80

samaccountname                 : krbtgt
memberof                       : {CN=Denied RODC Password Replication Group,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
ServicePrincipalName           : kadmin/changepw

samaccountname                 : sqlqa
memberof                       : {CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
ServicePrincipalName           : MSSQL_svc_qa/inlanefreight.local:1443

samaccountname                 : sql-test
ServicePrincipalName           : MSSQL_svc_test/inlanefreight.local:1443

samaccountname                 : sqlprod
memberof                       : {CN=Protected Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
ServicePrincipalName           : MSSQLSvc/sql01:1433
```

Finally, we can enumerate any users from other (foreign) domains with group membership within any groups in our current domain. We can see that the user `harry.jones` from the `FREIGHTLOGISTICS.LOCAL` domain is in our current domain's `administrators` group. If we compromise the current domain, we may obtain credentials for this user from the NTDS database and authenticate into the `FREIGHTLOGISTICS.LOCAL` domain.

```powershell-session
PS C:\htb> Find-ForeignGroup

GroupDomain             : INLANEFREIGHT.LOCAL
GroupName               : Administrators
GroupDistinguishedName  : CN=Administrators,CN=Builtin,DC=INLANEFREIGHT,DC=LOCAL
MemberDomain            : INLANEFREIGHT.LOCAL
MemberName              : S-1-5-21-888139820-103978830-333442103-1602
MemberDistinguishedName : CN=S-1-5-21-888139820-103978830-333442103-1602,CN=ForeignSecurityPrincipals,DC=INLANEFREIGHT,
                          DC=LOCAL
```

```powershell-session
PS C:\htb> Convert-SidToName S-1-5-21-888139820-103978830-333442103-1602

FREIGHTLOGISTIC\harry.jones
```

Another useful command is checking for users with Service Principal Names (SPNs) set in other domains that we can authenticate into via inbound or bi-directional trust relationships with forest-wide authentication allowing all users to authenticate across a trust or selective-authentication set up which allows specific users to authenticate. Here we can see one account in the `FREIGHTLOGISTICS.LOCAL` domain, which could be leveraged to Kerberoast across the forest trust.

```powershell-session
PS C:\htb> Get-DomainUser -SPN -Domain freightlogistics.local | select samaccountname,memberof,serviceprincipalname | fl

samaccountname       : krbtgt
memberof             : CN=Denied RODC Password Replication Group,CN=Users,DC=freightlogistics,DC=local
serviceprincipalname : kadmin/changepw

samaccountname       : svc_azure
memberof             : CN=Account Operators,CN=Builtin,DC=freightlogistics,DC=local
serviceprincipalname : freightlogistics/azureconnect:443
```

---

## Password Set Times

Analyzing the Password Set times is incredibly important when performing password sprays. Organizations are much more likely to find an automated password spray across all accounts than at a few guesses towards a small group of accounts.

- If you see a `several passwords set at the same time`, this indicates they were set by the Help Desk and may be the same. Because of Password Lockout Policies, you may not be able to exceed four failed passwords in fifteen minutes. However, if you think the password is the same across 20 accounts, for one user, you can guess passwords along the line of "Password2020" for a different use, you can use the company name like "Freight2020!".
    
- Additionally, if you see the password was set in July of 2019; then you can normally exclude "2020" from your password guessing and probably shouldn't guess variations that wouldn't make sense, such as "Winter2019."
    
- If you see an `old password that was set 2 years ago`, chances are this password is weak and also one of the first accounts I would recommend guessing the password to before launching a large Password Spray.
    
- In most organizations, administrators have multiple accounts. If you see the administrator changing his "user account" around the same time as his "Administrator Account", they are highly likely to use the same password for both accounts.
    

The following command will display all password set times.

```powershell-session
PS C:\htb> Get-DomainUser -Properties samaccountname,pwdlastset,lastlogon -Domain InlaneFreight.local | select samaccountname, pwdlastset, lastlogon | Sort-Object -Property pwdlastset
```

If you want only to show passwords set before a certain date:

```powershell-session
PS C:\htb> Get-DomainUser -Properties samaccountname,pwdlastset,lastlogon -Domain InlaneFreight.local | select samaccountname, pwdlastset, lastlogon | where { $_.pwdlastset -lt (Get-Date).addDays(-90) }
```

Blue team tip: Whenever you deal with a compromise or complete a Penetration Test. It is always a good idea to use the above command to verify all passwords have been rotated! You should never have passwords older than a year in your Active Directory.