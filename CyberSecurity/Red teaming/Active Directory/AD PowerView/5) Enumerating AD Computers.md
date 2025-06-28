Now that we have gathered user and group information, we need to find out information about the various hosts our target users can log in to, and if gaining SYSTEM access on any given host will open up different attack paths.

---

## Domain Computer Information

We can use the `Get-DomainComputer` function to enumerate many details about domain computers.

Enumerating AD Computers

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainComputer -Help

Get_DomainComputer -Identity <String[]> -SamAccountName <String[]> -Unconstrained <Boolean> -TrustedToAuth <Boolean> -Printers <Boolean> -SPN <String> -ServicePrincipalName <String> -OperatingSystem <String> -ServicePack <String> -SiteName <String> -Ping <Boolean> -Domain <String> -LDAPFilter <String> -Filter <String> -Properties <String[]> -SearchBase <String> -ADSPath <String> -Server <String> -DomainController <String> -SearchScope <SearchScope> -ResultPageSize <Int32> -ServerTimeLimit <Nullable`1> -SecurityMasks <Nullable`1> -Tombstone <Boolean> -FindOne <Boolean> -ReturnOne <Boolean> -Credential <NetworkCredential> -Raw <Boolean> -UACFilter <UACEnum>
```

Some of the most useful information we can gather is the hostname, operating system, and User Account Control (UAC) attributes.

Enumerating AD Computers

```powershell-session
PS C:\htb>.\SharpView.exe Get-DomanComputer -Properties dnshostname,operatingsystem,lastlogontimestamp,useraccountcontrol

[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainComputer] Get-DomainComputer filter string: (&(samAccountType=805306369))
useraccountcontrol             : SERVER_TRUST_ACCOUNT, TRUSTED_FOR_DELEGATION
lastlogontimestamp             : 8/17/2020 6:43:25 AM
dnshostname                    : DC01.INLANEFREIGHT.LOCAL
operatingsystem                : Windows Server 2016 Standard

useraccountcontrol             : WORKSTATION_TRUST_ACCOUNT
lastlogontimestamp             : 8/15/2020 9:49:12 PM
dnshostname                    : EXCHG01.INLANEFREIGHT.LOCAL
operatingsystem                : Windows Server 2016 Standard

useraccountcontrol             : WORKSTATION_TRUST_ACCOUNT, TRUSTED_TO_AUTH_FOR_DELEGATION
lastlogontimestamp             : 8/15/2020 7:42:00 PM
dnshostname                    : SQL01.INLANEFREIGHT.LOCAL
operatingsystem                : Windows Server 2016 Standard

useraccountcontrol             : WORKSTATION_TRUST_ACCOUNT
lastlogontimestamp             : 8/15/2020 5:55:24 PM
dnshostname                    : WS01.INLANEFREIGHT.LOCAL
operatingsystem                : Windows Server 2016 Standard

useraccountcontrol             : ACCOUNTDISABLE, WORKSTATION_TRUST_ACCOUNT
lastlogontimestamp             : 7/26/2020 9:58:15 PM
dnshostname                    : DC02.INLANEFREIGHT.LOCAL
```

Let's save this data to a CSV for our records using `PowerView`.

Enumerating AD Computers

```powershell-session
PS C:\htb> Get-DomainComputer -Properties dnshostname,operatingsystem,lastlogontimestamp,useraccountcontrol | Export-Csv .\inlanefreight_computers.csv -NoTypeInformation
```

---

## Finding Exploitable Machines
The most obvious thing in the above screenshot is within the "User Account Control" setting, and we will get into that shortly. However, tools like `Bloodhound` will quickly point this setting out, and it may become uncommon to find in organizations that have regular penetration tests performed. The following flags can be combined to help come up with attacks:

- `LastLogonTimeStamp`: This field exists to let administrators find stale machines. If this field is 90 days old for a machine, it has not been turned on and is missing both operating system and application patches. Due to this, administrators may want to automatically disable machines upon this field hitting 90 days of age. Attackers can use this field in combination with other fields such as `Operating System` or `When Created` to identify targets.
    
- `OperatingSystem`: This lists the Operating System. The obvious attack path is to find a Windows 7 box that is still active (LastLogonTimeStamp) and try attacks like Eternal Blue. Even if Eternal Blue is not applicable, older versions of Windows are ideal spots to work from as there are fewer logging/antivirus capabilities on older Windows. It's also important to know the differences between flavors of Windows. For example, Windows 10 Enterprise is the only version that comes with "Credential Guard" (Prevents Mimikatz from Stealing Passwords) Enabled by default. If you see Administrators logging into Windows 10 Professional and Windows 10 Enterprise, the Professional box should be targeted.
    
- `WhenCreated`: This field is created when a machine joins Active Directory. The older the box is, the more likely it is to deviate from the "Standard Build." Old workstations could have weaker local administration passwords, more local admins, vulnerable software, more data, etc.
---

## Computer Attacks
We can see if any computers in the domain are configured to allow [unconstrained delegation](https://adsecurity.org/?p=1667) and find one, the domain controller, which is standard.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainComputer -Unconstrained -Properties dnshostname,useraccountcontrol

[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainComputer] Searching for computers with for unconstrained delegation
[Get-DomainComputer] Get-DomainComputer filter string: (&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.
4.803:=524288))
useraccountcontrol             : SERVER_TRUST_ACCOUNT, TRUSTED_FOR_DELEGATION
dnshostname                    : DC01.INLANEFREIGHT.LOCAL
```

Finally, we can check for any hosts set up to allow for [constrained delegation](https://docs.microsoft.com/en-us/windows-server/security/kerberos/kerberos-constrained-delegation-overview#:~:text=Constrained%20delegation%20gives%20service%20administrators,to%20their%20back%2Dend%20services.).

```powershell-session
PS C:\htb> Get-DomainComputer -TrustedToAuth | select -Property dnshostname,useraccountcontrol

dnshostname                                                        useraccountcontrol
-----------                                                        ------------------
EXCHG01.INLANEFREIGHT.LOCAL                                 WORKSTATION_TRUST_ACCOUNT
SQL01.INLANEFREIGHT.LOCAL   WORKSTATION_TRUST_ACCOUNT, TRUSTED_TO_AUTH_FOR_DELEGATION
```

