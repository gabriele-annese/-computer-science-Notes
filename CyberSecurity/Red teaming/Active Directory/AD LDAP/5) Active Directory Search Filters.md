It is important to know how to build proper filter syntax for querying Active Directory using `PowerShell`. This knowledge gives us a deeper understanding of how our tools such as `PowerView` function under the hood and how we can further harness their power when enumerating Active Directory. It is also useful to understand how to formulate filters if you find yourself in a situation during an assessment without any of your tools available to you. Armed with this knowledge, you will be able to effectively "live off the land" and utilize built-in PowerShell cmdlets to perform your enumeration tasks (albeit slower than using many of the tools we will cover in this module).

---

## PowerShell Filters
Filters in PowerShell allow you to process piped output more efficiently and retrieve exactly the information you need from a command. Filters can be used to narrow down specific data in a large result or retrieve data that can then be piped to another command.

We can use filters with the `Filter` parameter. A basic example is querying a computer for installed software:

#### PowerShell - Filter Installed Software

  Active Directory Search Filters

```powershell
PS C:\htb> get-ciminstance win32_product | fl


IdentifyingNumber : {7FED75A1-600C-394B-8376-712E2A8861F2}
Name              : Microsoft Visual C++ 2017 x86 Additional Runtime - 14.12.25810
Vendor            : Microsoft Corporation
Version           : 14.12.25810
Caption           : Microsoft Visual C++ 2017 x86 Additional Runtime - 14.12.25810

IdentifyingNumber : {748D3A12-9B82-4B08-A0FF-CFDE83612E87}
Name              : VMware Tools
Vendor            : VMware, Inc.
Version           : 10.3.2.9925305
Caption           : VMware Tools

IdentifyingNumber : {EA8CB806-C109-4700-96B4-F1F268E5036C}
Name              : Local Administrator Password Solution
Vendor            : Microsoft Corporation
Version           : 6.2.0.0
Caption           : Local Administrator Password Solution

IdentifyingNumber : {2CD849A7-86A1-34A6-B8F9-D72F5B21A9AE}
Name              : Microsoft Visual C++ 2017 x64 Additional Runtime - 14.12.25810
Vendor            : Microsoft Corporation
Version           : 14.12.25810
Caption           : Microsoft Visual C++ 2017 x64 Additional Runtime - 14.12.25810

<SNIP>
```

The above command can provide considerable output. We can use the `Filter` parameter with the `notlike` operator to filter out all Microsoft software (which may be useful when enumerating a system for local privilege escalation vectors).

#### PowerShell - Filter Out Microsoft Software

```powershell
PS C:\htb> get-ciminstance win32_product -Filter "NOT Vendor like '%Microsoft%'" | fl


IdentifyingNumber : {748D3A12-9B82-4B08-A0FF-CFDE83612E87}
Name              : VMware Tools
Vendor            : VMware, Inc.
Version           : 10.3.2.9925305
Caption           : VMware Tools
```

---

## Operators
The `Filter` parameter requires at least one operator, which can help narrow down search results or reduce a large amount of command output to something more digestible. Filtering properly is important, especially when enumerating large environments and looking for very specific information in the command output. The following operators can be used with the `Filter` parameter:

|**Filter**|**Meaning**|
|---|---|
|-eq|Equal to|
|-le|Less than or equal to|
|-ge|Greater than or equal to|
|-ne|Not equal to|
|-lt|Less than|
|-gt|Greater than|
|-approx|Approximately equal to|
|-bor|Bitwise OR|
|-band|Bitwise AND|
|-recursivematch|Recursive match|
|-like|Like|
|-notlike|Not like|
|-and|Boolean AND|
|-or|Boolean OR|
|-not|Boolean NOT|

---

## Filter Examples: AD Object Properties
The filter can be used with operators to compare, exclude, search for, etc., a variety of AD object properties. Filters can be wrapped in curly braces, single quotes, parentheses, or double-quotes. For example, the following simple search filter using `Get-ADUser` to find information about the user `Sally Jones` can be written as follows:

#### PowerShell - Filter Examples

```powershell
Get-ADUser -Filter "name -eq 'sally jones'"
Get-ADUser -Filter {name -eq 'sally jones'}
Get-ADUser -Filter 'name -eq "sally jones"'
```

As seen above, the property value (here, `sally jones`) can be wrapped in single or double-quotes. The asterisk (`*`) can be used as a [wildcard](https://ss64.com/ps/syntax-wildcards.html) when performing queries. The command `Get-ADUser -filter {name -like "joe*"}` using a wildcard would return all domain users whose name start with `joe` (joe, joel, etc.). When using filters, certain characters must be escaped:

|**Character**|**Escaped As**|**Note**|
|---|---|---|
|“|`”|Only needed if the data is enclosed in double-quotes.|
|‘|\’|Only needed if the data is enclosed in single quotes.|
|NUL|\00|Standard LDAP escape sequence.|
|\|\5c|Standard LDAP escape sequence.|
|*|\2a|Escaped automatically, but only in -eq and -ne comparisons. Use -like and -notlike operators for wildcard comparison.|
|(|/28|Escaped automatically.|
|)|/29|Escaped automatically.|
|/|/2f|Escaped automatically.|
Let's try out some of these filters to enumerate the `INLANEFREIGHT.LOCAL` domain. We can search all domain computers for interesting hostnames. SQL servers are a particularly juicy target on internal assessments. The below command searches all hosts in the domain using `Get-ADComputer`, filtering on the `DNSHostName` property that contains the word `SQL`.

#### PowerShell - Filter For SQL

```powershell
PS C:\htb> Get-ADComputer  -Filter "DNSHostName -like 'SQL*'"

DistinguishedName : CN=SQL01,OU=SQL Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL
DNSHostName       : SQL01.INLANEFREIGHT.LOCAL
Enabled           : True
Name              : SQL01
ObjectClass       : computer
ObjectGUID        : 42cc9264-1655-4bfa-b5f9-21101afb33d0
SamAccountName    : SQL01$
SID               : S-1-5-21-2974783224-3764228556-2640795941-1104
UserPrincipalName :
```

Next, let's search for administrative groups. We can do this by filtering on the `adminCount` attribute. The group with this attribute set to `1` are protected by [AdminSDHolder](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory) and known as protected groups. `AdminSDHolder` is owned by the Domain Admins group. It has the privileges to change the permissions of objects in Active Directory. As discussed above, we can pipe the filtered command output and select just the group names.

#### PowerShell - Filter Administrative Groups

```powershell
PS C:\htb> Get-ADGroup -Filter "adminCount -eq 1" | select Name

Name
----
Administrators
Print Operators
Backup Operators
Replicator
Domain Controllers
Schema Admins
Enterprise Admins
Domain Admins
Server Operators
Account Operators
Read-only Domain Controllers
Security Operations
```

We can also combine filters. Let's search for all administrative users with the `DoesNotRequirePreAuth` attribute set, meaning that they can be ASREPRoasted (this attack will be covered in-depth in later modules). Here we are selecting all domain users and specifying two conditions with the `-eq` operator.

#### PowerShell - Filter Administrative Users

```powershell
PS C:\htb> Get-ADUser -Filter {adminCount -eq '1' -and DoesNotRequirePreAuth -eq 'True'}


DistinguishedName : CN=Jenna Smith,OU=Server Team,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
GivenName         : jenna
Name              : Jenna Smith
ObjectClass       : user
ObjectGUID        : ea3c930f-aa8e-4fdc-987c-4a9ee1a75409
SamAccountName    : jenna.smith
SID               : S-1-5-21-2974783224-3764228556-2640795941-1999
Surname           : smith
UserPrincipalName : jenna.smith@inlanefreight
```

Finally, let's see an example of combining filters and piping output multiple times to find our desired information. The following command can be used to find all administrative users with the "`servicePrincipalName`" attribute set, meaning that they can likely be subject to a Kerberoasting attack. This example applies the `Filter` parameter to find accounts with the `adminCount` attribute set to `1`, pipes this output to find all accounts with a Service Principal Name (SPN), and finally selects a few attributes about the accounts, including the account name, group membership, and the SPN.

#### PowerShell - Find Administrative Users with the ServicePrincipalName

```powershell
PS C:\htb> Get-ADUser -Filter "adminCount -eq '1'" -Properties * | where servicePrincipalName -ne $null | select SamAccountName,MemberOf,ServicePrincipalName | fl


SamAccountName       : krbtgt
MemberOf             : {CN=Denied RODC Password Replication Group,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
ServicePrincipalName : {kadmin/changepw}

SamAccountName       : sqlqa
MemberOf             : {CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
ServicePrincipalName : {MSSQL_svc_qa/inlanefreight.local:1443}
```

It would take an extremely long time to enumerate an Active Directory environment using many combinations of the commands above. This last example could be performed quickly and easily with tools such as `PowerView` or `Rubeus`. Nevertheless, it is important to apply filters competently when enumerating AD as the output from tools like `PowerView` can even be further filtered to provide us with precise results.