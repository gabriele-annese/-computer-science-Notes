
[`Lightweight Directory Access Protocol` (`LDAP`)](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol) is an integral part of Active Directory (AD). The latest LDAP specification is Version 3, which is published as [RFC 4511](https://tools.ietf.org/html/rfc4511). A firm understanding of how LDAP works in an AD environment is crucial for both attackers and defenders.

`LDAP` is an open-source and cross-platform protocol used for authentication against various directory services (such as AD). As discussed in the previous section, AD stores user account information and security information such as passwords and facilitates sharing this information with other devices on the network. `LDAP` is the language that applications use to communicate with other servers that also provide directory services. In other words, `LDAP` is a way that systems in the network environment can "speak" to AD.

An `LDAP` session begins by first connecting to an `LDAP` server, also known as a `Directory System Agent`. The Domain Controller in AD actively listens for `LDAP` requests, such as security authentication requests.

![image](https://academy.hackthebox.com/storage/modules/22/NEW_LDAP_auth.png)

The relationship between AD and `LDAP` can be compared to Apache and HTTP. The same way Apache is a web server that uses the HTTP protocol, Active Directory is a directory server that uses the `LDAP` protocol.

While uncommon, you may come across organizations while performing an assessment that does not have AD but does have LDAP, meaning that they most likely use another type of `LDAP` server such as [OpenLDAP](https://en.wikipedia.org/wiki/OpenLDAP).

---

## AD LDAP Authentication
`LDAP` is set up to authenticate credentials against AD using a "BIND" operation to set the authentication state for an `LDAP` session. There are two types of `LDAP` authentication.

1. **Simple Authentication:** This includes anonymous authentication, unauthenticated authentication, and username/password authentication. Simple authentication means that a username and password create a BIND request to authenticate to the LDAP server.
    
2. **SASL Authentication:** The [Simple Authentication and Security Layer (SASL)](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) framework uses other authentication services, such as Kerberos, to bind to the `LDAP` server and then uses this authentication service (Kerberos in this example) to authenticate to `LDAP`. The `LDAP` server uses the `LDAP` protocol to send an `LDAP` message to the authorization service which initiates a series of challenge/response messages resulting in either successful or unsuccessful authentication. SASL can provide further security due to the separation of authentication methods from application protocols.
    

LDAP authentication messages are sent in cleartext by default so anyone can sniff out LDAP messages on the internal network. It is recommended to use TLS encryption or similar to safeguard this information in transit.

---

## LDAP Queries
We can communicate with the directory service using `LDAP` queries to ask the service for information. For example, the following query can be used to find all workstations in a network `(objectCategory=computer)` while this query can be used to find all domain controllers: `(&(objectCategory=Computer)(userAccountControl:1.2.840.113556.1.4.803:=8192))`.

LDAP queries can be used to perform user-related searches, such as "`(&(objectCategory=person)(objectClass=user))`" which searches for all users, as well as group related searches such as "`(objectClass=group)`" which returns all groups. Here is one example of a simple query to find all AD groups using the "`Get-ADObject`" cmdlet and the "`LDAPFilter parameter`".

#### LDAP Query - User Related Search

```powershell
PS C:\htb> Get-ADObject -LDAPFilter '(objectClass=group)' | select name

name
--
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

<SNIP>
```

We can also use LDAP queries to perform more detailed searches. This query searches the domain for all administratively disabled accounts.

#### LDAP Query - Detailed Search

```powershell
PS C:\htb> Get-ADObject -LDAPFilter '(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))' -Properties * | select samaccountname,useraccountcontrol

samaccountname                                                         useraccountcontrol
--------------                                                         ------------------
Guest                ACCOUNTDISABLE, PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
DefaultAccount       ACCOUNTDISABLE, PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
krbtgt                               ACCOUNTDISABLE, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
caroline.ali                               ACCOUNTDISABLE, PASSWD_NOTREQD, NORMAL_ACCOUNT
$SH2000-FPNHUU487JP0                       ACCOUNTDISABLE, PASSWD_NOTREQD, NORMAL_ACCOUNT
SM_00390f38b41e488ab                                       ACCOUNTDISABLE, NORMAL_ACCOUNT
SM_e081bc60d79c4597b                                       ACCOUNTDISABLE, NORMAL_ACCOUNT
SM_a9a4eed7ad2d4369a                                       ACCOUNTDISABLE, NORMAL_ACCOUNT
SM_d836f82078bf4cf89                                       ACCOUNTDISABLE, NORMAL_ACCOUNT
SM_6a24f488535649558                                       ACCOUNTDISABLE, NORMAL_ACCOUNT
SM_08a2324990674a87b                                       ACCOUNTDISABLE, NORMAL_ACCOUNT
SM_d1fea2710dc146b1b                                       ACCOUNTDISABLE, NORMAL_ACCOUNT
SM_b56189681baa441db                                       ACCOUNTDISABLE, NORMAL_ACCOUNT
SM_b72a918d27554863b                                       ACCOUNTDISABLE, NORMAL_ACCOUNT
```

More examples of basic and more advanced `LDAP` queries for AD can be found at the following links:

- LDAP queries related to AD [computers](https://ldapwiki.com/wiki/Wiki.jsp?page=Active%20Directory%20Computer%20Related%20LDAP%20Query)
- LDAP queries related to AD [users](https://ldapwiki.com/wiki/Wiki.jsp?page=Active%20Directory%20User%20Related%20Searches)
- LDAP queries related to AD [groups](https://ldapwiki.com/wiki/Wiki.jsp?page=Active%20Directory%20Group%20Related%20Searches)

`LDAP` queries are extremely powerful tools for querying Active Directory. We can harness their power to gather a wide variety of information, map out the AD environment, and hunt for misconfigurations. LDAP queries can be combined with filters to perform even more granular searches. The next two sections will cover both AD and LDAP search filters in-depth to prepare us for introducing a variety of AD enumeration tools in subsequent modules.

