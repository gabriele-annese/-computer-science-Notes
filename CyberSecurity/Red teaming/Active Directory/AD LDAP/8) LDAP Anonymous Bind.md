# DAP Anonymous Bind

Lightweight Directory Access Protocol (LDAP) is a protocol that is used for accessing directory services.

---

## Leveraging LDAP Anonymous Bind

LDAP anonymous binds allow unauthenticated attackers to retrieve information from the domain, such as a full listing of users, groups, computers, user account attributes, and the domain password policy. Linux hosts running open-source versions of LDAP and Linux vCenter appliances are often configured to allow anonymous binds.

When an LDAP server allows anonymous base binds, an attacker does not need to know a base object to query a considerable amount of information from the domain. This can also be leveraged to mount a password spraying attack or read information such as passwords stored in account description fields. Tools such as [windapsearch](https://github.com/ropnop/windapsearch) and [ldapsearch](https://linux.die.net/man/1/ldapsearch) can be utilized to enumerate domain information via an anonymous LDAP bind. Information that we obtain from an anonymous LDAP bind can be leveraged to mount a password spraying or AS-REPRoasting attack, read information such as passwords stored in account description fields.

We can use `Python` to quickly check if we can interact with LDAP without credentials.

Code: python

```python
Python 3.8.5 (default, Aug  2 2020, 15:09:07) 
[GCC 10.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from ldap3 import *
>>> s = Server('10.129.1.207',get_info = ALL)
>>> c =  Connection(s, '', '')
>>> c.bind()
True
>>> s.info
DSA info (from DSE):
  Supported LDAP versions: 3, 2
  Naming contexts: 
    DC=INLANEFREIGHT,DC=LOCAL
    CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
    CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
    DC=DomainDnsZones,DC=INLANEFREIGHT,DC=LOCAL
    DC=ForestDnsZones,DC=INLANEFREIGHT,DC=LOCAL
  Supported controls: 
    
	<SNIP>
	
  dnsHostName: 
    DC01.INLANEFREIGHT.LOCAL
  ldapServiceName: 
    INLANEFREIGHT.LOCAL:dc01$@INLANEFREIGHT.LOCAL
  serverName: 
    CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
  isSynchronized: 
    TRUE
  isGlobalCatalogReady: 
    TRUE
  domainFunctionality: 
    7
  forestFunctionality: 
    7
  domainControllerFunctionality: 
    7
```

---

## Using Ldapsearch

We can confirm anonymous LDAP bind with `ldapsearch` and retrieve all AD objects from LDAP.

  LDAP Anonymous Bind

```shell-session
BusySec@htb[/htb]$ ldapsearch -H ldap://10.129.1.207 -x -b "dc=inlanefreight,dc=local"
# extended LDIF
#
# LDAPv3
# base <dc=inlanefreight,dc=local> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# INLANEFREIGHT.LOCAL
dn: DC=INLANEFREIGHT,DC=LOCAL
objectClass: top
objectClass: domain
objectClass: domainDNS
distinguishedName: DC=INLANEFREIGHT,DC=LOCAL
instanceType: 5
whenCreated: 20200726201343.0Z
whenChanged: 20200827025341.0Z
subRefs: DC=LOGISTICS,DC=INLANEFREIGHT,DC=LOCAL
subRefs: DC=ForestDnsZones,DC=INLANEFREIGHT,DC=LOCAL
subRefs: DC=DomainDnsZones,DC=INLANEFREIGHT,DC=LOCAL
subRefs: CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
```

## Using Windapsearch

`Windapsearch` is a Python script used to perform anonymous and authenticated LDAP enumeration of AD users, groups, and computers using LDAP queries. It is an alternative to tools such as `ldapsearch`, which require you to craft custom LDAP queries. We can use it to confirm LDAP NULL session authentication but providing a blank username with `-u ""` and add `--functionality` to confirm the domain functional level.

  LDAP Anonymous Bind

```shell-session
BusySec@htb[/htb]$ python3 windapsearch.py --dc-ip 10.129.1.207 -u "" --functionality
[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 10.129.1.207
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Functionality Levels:
[+]	 domainFunctionality: 2016
[+]	 forestFunctionality: 2016
[+]	 domainControllerFunctionality: 2016
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 None

[*] Bye!
```

We can pull a listing of all domain users to use in a password spraying attack.

  LDAP Anonymous Bind

```shell-session
BusySec@htb[/htb]$ python3 windapsearch.py --dc-ip 10.129.1.207 -u "" -U
[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 10.129.1.207
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 None

[+] Enumerating all AD users
[+]	Found 1024 users: 

cn: Guest
cn: DefaultAccount
cn: LOGISTICS$
cn: sqldev
cn: sqlprod
cn: svc-scan

<SNIP>
```

We can obtain information about all domain computers.

  LDAP Anonymous Bind

```shell-session
BusySec@htb[/htb]$ python3 windapsearch.py --dc-ip 10.129.1.207 -u "" -C
[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 10.129.1.207
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 None

[+] Enumerating all AD computers
[+]	Found 5 computers: 

cn: DC01
operatingSystem: Windows Server 2016 Standard
operatingSystemVersion: 10.0 (14393)
dNSHostName: DC01.INLANEFREIGHT.LOCAL

cn: EXCHG01
operatingSystem: Windows Server 2016 Standard
operatingSystemVersion: 10.0 (14393)
dNSHostName: EXCHG01.INLANEFREIGHT.LOCAL

cn: SQL01
operatingSystem: Windows Server 2016 Standard
operatingSystemVersion: 10.0 (14393)
dNSHostName: SQL01.INLANEFREIGHT.LOCAL

cn: WS01
operatingSystem: Windows Server 2016 Standard
operatingSystemVersion: 10.0 (14393)
dNSHostName: WS01.INLANEFREIGHT.LOCAL

cn: DC02
dNSHostName: DC02.INLANEFREIGHT.LOCAL

[*] Bye!
```

This process can be repeated to pull group information and more detailed information such as unconstrained users and computers, GPO information, user and computer attributes, and more.

---

## Other Tools

There are many other tools and helper scripts for retrieving information from LDAP. This script [ldapsearch-ad.py](https://github.com/yaap7/ldapsearch-ad) is similar to `windapsearch`.

  LDAP Anonymous Bind

```shell-session
BusySec@htb[/htb]$ python3 ldapsearch-ad.py -h
usage: ldapsearch-ad.py [-h] -l LDAP_SERVER [-ssl] -t REQUEST_TYPE [-d DOMAIN] [-u USERNAME] [-p PASSWORD]
                        [-s SEARCH_FILTER] [-z SIZE_LIMIT] [-o OUTPUT_FILE] [-v]
                        [search_attributes [search_attributes ...]]

Active Directory LDAP Enumerator

positional arguments:
  search_attributes     LDAP attributes to look for (default is all).

optional arguments:
  -h, --help            show this help message and exit
  -l LDAP_SERVER, --server LDAP_SERVER
                        IP address of the LDAP server.
  -ssl, --ssl           Force an SSL connection?.
  -t REQUEST_TYPE, --type REQUEST_TYPE
                        Request type: info, whoami, search, search-large, trusts, pass-pols, show-admins,
                        show-user, show-user-list, kerberoast, all
  -d DOMAIN, --domain DOMAIN
                        Authentication account's FQDN. Example: "contoso.local".
  -u USERNAME, --username USERNAME
                        Authentication account's username.
  -p PASSWORD, --password PASSWORD
                        Authentication account's password.
  -s SEARCH_FILTER, --search-filter SEARCH_FILTER
                        Search filter (use LDAP format).
  -z SIZE_LIMIT, --size_limit SIZE_LIMIT
                        Size limit (default is 100, or server' own limit).
  -o OUTPUT_FILE, --output OUTPUT_FILE
                        Write results in specified file too.
  -v, --verbose         Turn on debug mode
```

We can use it to pull domain information and confirm a NULL bind. This particular tool requires valid domain user credentials to perform additional enumeration.

  LDAP Anonymous Bind

```shell-session
BusySec@htb[/htb]$ python3 ldapsearch-ad.py -l 10.129.1.207 -t info

### Server infos ###
[+] Forest functionality level = Windows 2016
[+] Domain functionality level = Windows 2016
[+] Domain controller functionality level = Windows 2016
[+] rootDomainNamingContext = DC=INLANEFREIGHT,DC=LOCAL
[+] defaultNamingContext = DC=INLANEFREIGHT,DC=LOCAL
[+] ldapServiceName = INLANEFREIGHT.LOCAL:dc01$@INLANEFREIGHT.LOCAL
[+] naming_contexts = ['DC=INLANEFREIGHT,DC=LOCAL', 'CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL', 'CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL', 'DC=DomainDnsZones,DC=INLANEFREIGHT,DC=LOCAL', 'DC=ForestDnsZones,DC=INLANEFREIGHT,DC=LOCAL']
```


