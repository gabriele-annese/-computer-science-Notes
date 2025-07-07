[SharpHound](https://github.com/BloodHoundAD/SharpHound) is the official data collector tool for [BloodHound](https://github.com/BloodHoundAD/BloodHound), is written in C# and can be run on Windows systems with the .NET framework installed. The tool uses various techniques to gather data from Active Directory, including native Windows API functions and LDAP queries.

The data collected by SharpHound can be used to identify security weaknesses in an Active Directory environment to attack it or to plan for remediation.

This section will discover the basic functionalities of enumerating Active Directory using SharpHound and how to do it.

## Basic Enumeration

By default SharpHound, if run without any options, will identify the domain to which the user who ran it belongs and will execute the default collection. Let's execute SharpHound without any options.

**Note:** To follow the exercise start the target machine and connect via RDP with the following credentials `htb-student:HTBRocks!`.

#### Running SharpHound without any option

```cmd-session
C:\tools> SharpHound.exe
2023-01-10T09:10:27.5517894-06:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2023-01-10T09:10:27.6678232-06:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-10T09:10:27.6834781-06:00|INFORMATION|Initializing SharpHound at 9:10 AM on 1/10/2023
2023-01-10T09:11:12.0547392-06:00|INFORMATION|Flags: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-10T09:11:12.2081156-06:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.HTB
2023-01-10T09:11:12.2394159-06:00|INFORMATION|Producer has finished, closing LDAP channel
2023-01-10T09:11:12.2615280-06:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-01-10T09:11:42.6237001-06:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 35 MB RAM
2023-01-10T09:12:12.6416076-06:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 37 MB RAM
2023-01-10T09:12:42.9758511-06:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 37 MB RAM
2023-01-10T09:12:43.2077516-06:00|INFORMATION|Consumers finished, closing output channel
2023-01-10T09:12:43.2545768-06:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2023-01-10T09:12:43.3771345-06:00|INFORMATION|Status: 94 objects finished (+94 1.032967)/s -- Using 42 MB RAM
2023-01-10T09:12:43.3771345-06:00|INFORMATION|Enumeration finished in 00:01:31.1684392
2023-01-10T09:12:43.4617976-06:00|INFORMATION|Saving cache with stats: 53 ID to type mappings.
 53 name to SID mappings.
 1 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2023-01-10T09:12:43.4617976-06:00|INFORMATION|SharpHound Enumeration Completed at 9:12 AM on 1/10/2023! Happy Graphing!
```

The 2nd line in the output above indicates the collection method used by default: `Resolved Collection Methods: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote`. Those methods mean that we will collect the following:

1. Users and Computers.
2. Active Directory security group membership.
3. Domain trusts.
4. Abusable permissions on AD objects.
5. OU tree structure.
6. Group Policy links.
7. The most relevant AD object properties.
8. Local groups from domain-joined Windows systems and local privileges such as RDP, DCOM, and PSRemote.
9. User sessions.
10. All SPN (Service Principal Names).

To get the information for local groups and sessions, SharpHound will attempt to connect to each domain-joined Windows computer from the list of computers it collected. If the user from which SharpHound is running has privileges on the remote computer, it will collect the following information:

1. The members of the local administrators, remote desktop, distributed COM, and remote management groups.
2. Active sessions to correlate to systems where users are interactively logged on.

**Note:** Gathering information from domain-joined machines, such as local group membership and active sessions, is only possible if the user session from which SharpHound is being executed has Administrator rights on the target computer.

Once SharpHound terminates, by default, it will produce a zip file whose name starts with the current date and ends with BloodHound. This zip archive contains a group of JSON files:

![image](https://academy.hackthebox.com/storage/modules/69/bh_zip.png)

## Importing Data into BloodHound

1. Start the `neo4j` database service:

#### Start Service

```powershell-session
PS C:\htb> net start neo4j
The Neo4j Graph Database - neo4j service is starting..
The Neo4j Graph Database - neo4j service was started successfully.
```

2. Launch `C:\Tools\BloodHound\BloodHound.exe` and log in with the following credentials:

```shell-session
Username: neo4j
Password: Password123
```

![text](https://academy.hackthebox.com/storage/modules/69/BH_login.png)

3. Click the upload button on the far right, browse to the zip file, and upload it. You will see a status showing upload % completion.

![text](https://academy.hackthebox.com/storage/modules/69/bh_upload_data.jpg)

**Note:** We can upload as many zip files as we want. BloodHound will not duplicate the data but add data not present in the database.

4. Once the upload is complete, we can analyze the data. If we want to view information about the domain, we can type `Domain:INLANEFREIGHT.HTB` into the search box. This will show an icon with the domain name. If you click the icon, it will display information about the node (the domain), how many users, groups, computers, OUs, etc.

![text](https://academy.hackthebox.com/storage/modules/69/bh_node_domain.jpg)

5. Now, we can start analyzing the information in the bloodhound and find the paths to our targets.