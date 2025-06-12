As with SMB, once we have domain credentials, we can extract a wide variety of information from LDAP, including user, group, computer, trust, GPO info, the domain password policy, etc. `ldapsearch-ad.py` and `windapsearch` are useful for performing this enumeration.

---

## Windapsearch

```shell
BusySec@htb[/htb]$ python3 windapsearch.py --dc-ip 10.129.1.207 -u inlanefreight\\james.cross --da

Password for inlanefreight\james.cross: 

[+] Using Domain Controller at: 10.129.1.207
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 u:INLANEFREIGHT\james.cross
[+] Attempting to enumerate all Domain Admins
[+] Using DN: CN=Domain Admins,CN=Users.CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]	Found 14 Domain Admins:

cn: Administrator
userPrincipalName: Administrator@INLANEFREIGHT.LOCAL

cn: daniel.carter
cn: sqlqa
cn: svc-backup
cn: svc-secops
cn: cliff.moore
cn: svc-ata
cn: svc-sccm
cn: mrb3n
cn: sarah.lafferty

cn: Harry Jones
userPrincipalName: harry.jones@inlanefreight

cn: pixis
cn: Cry0l1t3
cn: knightmare

[+] Using DN: CN=Domain Admins,CN=Users.CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]	Found 14 Domain Admins:

cn: Administrator
userPrincipalName: Administrator@INLANEFREIGHT.LOCAL

cn: daniel.carter
cn: sqlqa
cn: svc-backup
cn: svc-secops

<SNIP>
```

Some additional useful options, including pulling users and computers with unconstrained delegation.

```shell
BusySec@htb[/htb]$ python3 windapsearch.py --dc-ip 10.129.1.207 -d inlanefreight.local -u inlanefreight\\james.cross --unconstrained-users

Password for inlanefreight\james.cross: 

[+] Using Domain Controller at: 10.129.1.207
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 u:INLANEFREIGHT\james.cross
[+] Attempting to enumerate all user objects with unconstrained delegation
[+]	Found 1 Users with unconstrained delegation:

CN=sqldev,OU=Service Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL

[*] Bye!
```

---

## Ldapsearch-ad

This tool can perform all of the standard enumeration and a few built-in searches to simplify things. We can quickly obtain the password policy.

```shell
BusySec@htb[/htb]$ python3 ldapsearch-ad.py -l 10.129.1.207 -d inlanefreight -u james.cross -p Summer2020 -t pass-pols

### Result of "pass-pols" command ###
Default password policy:
[+] |___Minimum password length = 7
[+] |___Password complexity = Disabled
[*] |___Lockout threshold = Disabled
[+] No fine grained password policy found (high privileges are required).
```

We can look for users who may be subject to a Kerberoasting attack.

```shell
BusySec@htb[/htb]$ python3 ldapsearch-ad.py -l 10.129.1.207 -d inlanefreight -u james.cross -p Summer2020 -t kerberoast | grep servicePrincipalName:

    servicePrincipalName: CIFS/roguecomputer.inlanefreight.local
    servicePrincipalName: MSSQLSvc/sql01:1433
    servicePrincipalName: MSSQL_svc_qa/inlanefreight.local:1443
    servicePrincipalName: MSSQL_svc_test/inlanefreight.local:1443
    servicePrincipalName: IIS_dev/inlanefreight.local:80
```

Also, it quickly retrieves users that can be ASREPRoasted.

```shell
BusySec@htb[/htb]$ python3 ldapsearch-ad.py -l 10.129.1.207 -d inlanefreight -u james.cross -p Summer2020 -t asreproast

### Result of "asreproast" command ###
[*] DN: CN=Amber Smith,OU=Contractors,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL - STATUS: Read - READ TIME: 2020-09-02T17:11:45.572421
    cn: Amber Smith
    sAMAccountName: amber.smith

[*] DN: CN=Jenna Smith,OU=Server Team,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL - STATUS: Read - READ TIME: 2020-09-02T17:11:45.572729
    cn: Jenna Smith
    sAMAccountName: jenna.smith
```

---

## LDAP Wrap-up

We can use tools such as the two shown in this section to perform a considerable amount of AD enumeration using LDAP. The tools have many built-in queries to simplify searching and provide us with the most useful and actionable data. We can also combine these tools with the custom LDAP search filters that we learned about earlier in the module. These are great tools to keep in our arsenal, especially when we are in a position where most an AD assessment has to be performed from a Linux attack box.