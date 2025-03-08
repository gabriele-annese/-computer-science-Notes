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