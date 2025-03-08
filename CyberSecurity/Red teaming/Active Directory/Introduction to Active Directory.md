Active Directory (AD) is a directory service for Windows network environment. Its a distributed hierarchical structure that allow for centralized management of organization's resources. AD provides authentication and authorization functions within Windows domain environment. 

AD it's essentially a read-only database accessible to all users within domain, regardless of their privilege level.


## Structure of AD Network

*Active Directory Domain Service* (AD DS) store information about username password and manage the rights needed for authentication. 

> Note
> By default the settings are not secure specially for bug network

AD user without privilege can be enumerate the most majority objects:


| Domain Computers         | Domain Users                |
| ------------------------ | --------------------------- |
| Domain Group Information | Organizational Units (OUs)  |
| Default Domain Policy    | Functional Domain Levels    |
| Password Policy          | Group Policy Objects (GPOs) |
| Domain Trusts            | Access Control Lists (ACLs) |


### Forest
The forest in AD is the containers of AD domains. Each AD domains can be have sub-domains. A domain is a structure within which contained objects (users, computers, and groups) are accessible.

Example of simple AD network.

```shell-session
INLANEFREIGHT.LOCAL/
├── ADMIN.INLANEFREIGHT.LOCAL
│   ├── GPOs
│   └── OU
│       └── EMPLOYEES
│           ├── COMPUTERS
│           │   └── FILE01
│           ├── GROUPS
│           │   └── HQ Staff
│           └── USERS
│               └── barbara.jones
├── CORP.INLANEFREIGHT.LOCAL
└── DEV.INLANEFREIGHT.LOCAL
```

`INLANEFREIGHT.LOCAL` is the root domain and contains the sub-domains `ADMIN.INLANEFREIGHT.LOCAL` `CORP.INLANEFREIGHT`.LOCAL and `DEV.INLANEFREIGHT.LOCAL`.

### Trust relationship
It is typically see multiple domains (or forests) linked together via trust relationship in a organizations that perform a lot of acquisitions. Its often quicker and easier to create all new users in the current domain. This can be introduce a slew security problems if not appropriately administrated

This is a example how two forest are 
![[Pasted image 20250308115811.png]]

`INLANEFREIGHT.LOCAL` and `FREIGHTLOGISTICS.LOCAL` have a trust relationship, meaning that users in the `INLANEFREIGHT.LOCAL` can be access resources in the forest `FREIGHTLOGISTICS.LOCAL` and vice versa.

## AD Terminology

### Objects
an object is ANY resources in the AD environment such as OUs, printers, users, domain controllers, etc.

### Attributes
Every objects in AD env has an associated [attributes](https://learn.microsoft.com/en-us/windows/win32/adschema/attributes-all) used to define a characteristic of the given object. A computer object contains attributes such as the hostname and DNS name. All attributes in AD have an associated *LDAP* name that can be use to perform *LDAP queries*, such as `displayName` for `Full Name` and `given name` for `First Name`.

### Schema
The [Schema](https://learn.microsoft.com/en-us/windows/win32/ad/schema) is a *blueprint* of any enterprise environment. It defines what types of objects can exists in the AD database and their associated attributes. For example users in AD belong to the class "user" and computer object to class "computer". Each object has its own information (some required to be set and other optional) that are stored in Attributes. When a object is created from class this call *installation* , and object created from a specific class this called *instance*. For example if we take the computer *RDS01* this computer is a *instance* of *computer* class.

### Domain
a Domain is a logical group of object such OUs, computers, users, groups, etc. Domains can *operate entirely* independently of another or be connected via *trust relationship*.

### Forest
A forest is a collection of Active Directory domains. It is the topmost container and contains all of the AD objects introduced below, including but not limited to domains, users, groups, computers, and Group Policy objects. *A forest can contain one or multiple domains and be thought of as a state in the US or a country within the EU*. Each forest operates independently but may have various trust relationships with other forests.

### Tree
A tree is a collection of Active Directory domains that begins at a single root domain. A forest is a collection of AD trees. Each domain in a tree shares a boundary with the other domains. A parent-child trust relationship is formed when a domain is added under another domain in a tree. Two trees in the same forest cannot share a name (namespace). Let's say we have two trees in an AD forest: `inlanefreight.local` and `ilfreight.local`. A child domain of the first would be `corp.inlanefreight.local` while a child domain of the second could be `corp.ilfreight.local`. 

 > Note
 > *All domains in a tree share a standard Global Catalog which contains all information about objects that belong to the tree.*
 

### Container
A container objects hold other objects and have a define place in the directory subtree hierarchy.

### Leaf
A leaf objects do not contains other objects and are found to the end of the subtree directory.

### Global Unique Identifier (GUID)
A [GUID](https://docs.microsoft.com/en-us/windows/win32/adschema/a-objectguid) is a unique 128-bit value assigned when a domain user or group is created. This GUID value is unique across the enterprise, similar to a MAC address. 
*Every single object created by Active Directory is assigned a GUID*, not only user and group objects. The GUID is stored in the `ObjectGUID` attribute. When querying for an AD object (such as a user, group, computer, domain, domain controller, etc.), we can query for its `objectGUID` value using PowerShell or search for it by specifying its distinguished name, GUID, SID, or SAM account name. 
GUIDs are used by AD to identify objects internally. Searching in Active Directory by GUID value is probably the most accurate and reliable way to find the exact object you are looking for, especially if the global catalog may contain similar matches for an object name. Specifying the `ObjectGUID` value when performing AD enumeration will ensure that we get the most accurate results pertaining to the object we are searching for information about. 

The `ObjectGUID` property `never` changes and is associated with the object for as long as that object exists in the domain.


### Security Principals
[Security principals](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-principals) *are anything that the operating system can authenticate*, including users, computer accounts, or even threads/processes that run in the context of a user or computer account (i.e., an application such as Tomcat running in the context of a service account within the domain). 
In AD, *security principles are domain objects that can manage access to other resources within the domain*. We can also have local user accounts and security groups used to control access to resources on only that specific computer. These are not managed by AD but rather by the [Security Accounts Manager (SAM)](https://en.wikipedia.org/wiki/Security_Account_Manager).
#### Security Identifier (SID)
A [security identifier](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-principals), or SID is used as a unique identifier for a security principal or security group. *Every account, group, or process has its own unique SID*, which, in an AD environment, is issued by the domain controller and stored in a secure database. A SID can only be used once. 
Even if the security principle is deleted, it can never be used again in that environment to identify another user or group. 
When a user logs in, the system creates an `access token` for them which contains the user's SID, the rights they have been granted, and the SIDs for any groups that the user is a member of. 
This token is used to check rights whenever the user performs an action on the computer.

There are also [well-known SIDs](https://ldapwiki.com/wiki/Wiki.jsp?page=Well-known%20Security%20Identifiers) that are used to identify generic users and groups. These are the same across all operating systems. An example is the `Everyone` group.

### Distinguished Name (DN)
A [Distinguished Name (DN)](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ldap/distinguished-names) describes the full path to an object in AD (such as `cn=bjones, ou=IT, ou=Employees, dc=inlanefreight, dc=local`). 
In this example, the user `bjones` works in the IT department of the company `Inlanefreight`, and his account is created in an `Organizational Unit (OU)` that holds accounts for company employees. 
The Common Name (CN) `bjones` is just one way the user object could be searched for or accessed within the domain.

### Relative Distinguished Name (RDN)
A [Relative Distinguished Name (RDN)](https://docs.microsoft.com/en-us/windows/win32/ad/object-names-and-identities) is a single component of the Distinguished Name that identifies the object as unique from other objects at the current level in the naming hierarchy. 
In our example, `bjones` is the Relative Distinguished Name of the object. AD does not allow two objects with the same name under the same parent container, but there can be two objects with the same RDNs that are still unique in the domain because they have different DNs. For example, the object `cn=bjones,dc=dev,dc=inlanefreight,dc=local` would be recognized as different from `cn=bjones,dc=inlanefreight,dc=local`.

![[Pasted image 20250308125342.png]]

