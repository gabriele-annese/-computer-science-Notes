
Group Policy provides systems administrators with a centralized way to manage configuration settings and manage operating systems and user and computer settings in a Windows environment. A [Group Policy Object (GPO)](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/policy/group-policy-objects) is a collection of policy settings. GPOs include policies such as screen lock timeout, disabling USB ports, domain password policy, push out software, manage applications, and more. GPOs can be applied to individual users and hosts or groups by being applied directly to an Organizational Unit (OU). Gaining rights over a GPO can lead to lateral and vertical movement up to full domain compromise and can also be used as a persistence mechanism. Like ACLs, GPOs are often overlooked, and one misconfigured GPO can have catastrophic results.

We can use `Powerview`/`Sharpview`, `BloodHound`, and [Group3r](https://github.com/Group3r/Group3r) to enumerate Group Policy security misconfigurations. This section will show some of the enumeration techniques we can perform on the command line using `PowerView` and `SharpView`.

---

## GPO Abuse

GPOs can be abused to perform attacks such as adding additional rights to a user, adding a local admin, or creating an immediate scheduled task. There are several ways to gain persistence via GPOs:

- Configure a GPO to run any of the above attacks.
- Create a scheduled task to modify group membership, add an account, run DCSync, or send back a reverse shell connection.
- Install targeted malware across the entire Domain.

[SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) is an excellent tool that can be used to take advantage of GPO misconfigurations. This section will help arm us with the data that we need to use tools such as this.

---

## Gathering GPO Data

Let's start by gathering GPO names. In our test domain `INLANEFREIGHT.LOCAL`, there are 20 GPOs applied to various OUs.

```powershell-session
PS C:\htb> Get-DomainGPO | select displayname

displayname
-----------
Default Domain Policy
Default Domain Controllers Policy
LAPS Install
LAPS
Disable LM Hash
Disable CMD.exe
Disallow removable media
Prevent software installs
Disable guest account
Disable SMBv1
Map home drive
Disable Forced Restarts
Screensaver
Applocker
Fine-grained password policy
Restrict Control Panel
User - MS Office
User - Browser Settings
Audit Policy
PowerShell logging
```

We can also check which GPOs apply to a specific computer.

```powershell-session
PS C:\htb> Get-DomainGPO -ComputerName WS01 | select displayname

displayname
-----------
LAPS
Restrict Control Panel
Applocker
Disable Forced Restarts
Prevent software installs
Disallow removable media
Disable CMD.exe
Disable LM Hash
Disable Defender
PowerShell logging
Audit Policy
User - Browser Settings
User - MS Office
Fine-grained password policy
Screensaver
Map home drive
Disable SMBv1
Disable guest account
Default Domain Policy
```

Analyzing the GPO names can give us an idea of some of the security configurations in the target domain, such as LAPS, AppLocker, PowerShell Logging, cmd.exe disabled for workstations, etc. We can check for hosts/users that these GPOs are not applied to and plan out our attack paths for circumventing these controls.

If we do not have tools available to us, we can use [gpresult](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/gpresult), which is a built-in tool that determines GPOs that have been applied to a given user or computer and their settings. We can use specific commands to see the GPOs applied to a specific user and computer, respectively, such as:

```cmd-session
C:\> gpresult /r /user:harry.jones
```

```cmd-session
C:\> gpresult /r /S WS01
```

The tool can output in HTML format with a command such as `gpresult /h gpo_report.html`.

Let's use `gpresult` to see what GPOs are applied to a workstation in the domain.

```cmd-session
C:\htb> gpresult /r /S WS01

Microsoft (R) Windows (R) Operating System Group Policy Result tool v2.0
© 2016 Microsoft Corporation. All rights reserved.

Created on 8/27/2020 at 12:05:47 AM


RSOP data for INLANEFREIGHT\Administrator on WS01 : Logging Mode
-----------------------------------------------------------------

OS Configuration:            Member Server
OS Version:                  10.0.14393
Site Name:                   Default-First-Site-Name
Roaming Profile:             N/A
Local Profile:               C:\Users\administrator.INLANEFREIGHT
Connected over a slow link?: No


COMPUTER SETTINGS
------------------

    Last time Group Policy was applied: 8/26/2020 at 10:57:00 PM
    Group Policy was applied from:      DC01.INLANEFREIGHT.LOCAL
    Group Policy slow link threshold:   500 kbps
    Domain Name:                        INLANEFREIGHT
    Domain Type:                        Windows 2008 or later

    Applied Group Policy Objects
    -----------------------------
        LAPS
        LAPS
        Disable LM Hash
        Prevent software installs
        Default Domain Policy
        LAPS
        Disable LM Hash
        Prevent software installs
        Disable guest account

    The following GPOs were not applied because they were filtered out
    -------------------------------------------------------------------
        Local Group Policy
            Filtering:  Not Applied (Empty)

    The computer is a part of the following security groups
    -------------------------------------------------------
        BUILTIN\Administrators
        Everyone
        BUILTIN\Users
        NT AUTHORITY\NETWORK
        NT AUTHORITY\Authenticated Users
        This Organization
        Authentication authority asserted identity
        System Mandatory Level


USER SETTINGS
--------------
    CN=Administrator,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
    Last time Group Policy was applied: 7/30/2020 at 3:10:28 PM
    Group Policy was applied from:      N/A
    Group Policy slow link threshold:   500 kbps
    Domain Name:                        INLANEFREIGHT
    Domain Type:                        Windows 2008 or later

    Applied Group Policy Objects
    -----------------------------
        N/A

    The following GPOs were not applied because they were filtered out
    -------------------------------------------------------------------
        Local Group Policy
            Filtering:  Not Applied (Empty)

    The user is a part of the following security groups
    ---------------------------------------------------
        Domain Users
        Everyone
        BUILTIN\Users
        BUILTIN\Administrators
        NT AUTHORITY\INTERACTIVE
        CONSOLE LOGON
        NT AUTHORITY\Authenticated Users
        This Organization
        LOCAL
        Domain Admins
        Authentication authority asserted identity
        High Mandatory Level
```

---

## GPO Permissions

After reviewing all of the GPOs applied throughout the domain, it is always good to look at GPO permissions. We can use the `Get-DomainGPO` and `Get-ObjectAcl` using the SID for the `Domain Users` group to see if this group has any permissions assigned to any GPOs.

```powershell-session
PS C:\htb> Get-DomainGPO | Get-ObjectAcl | ? {$_.SecurityIdentifier -eq 'S-1-5-21-2974783224-3764228556-2640795941-513'}

ObjectDN              : CN={831DE3ED-40B1-4703-ABA7-8EA13B2EB118},CN=Policies,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ObjectSID             :
ActiveDirectoryRights : CreateChild, DeleteChild, ReadProperty, WriteProperty, GenericExecute
BinaryLength          : 36
AceQualifier          : AccessAllowed
IsCallback            : False
OpaqueLength          : 0
AccessMask            : 131127
SecurityIdentifier    : S-1-5-21-2974783224-3764228556-2640795941-513
AceType               : AccessAllowed
AceFlags              : ContainerInherit
IsInherited           : False
InheritanceFlags      : ContainerInherit
PropagationFlags      : None
AuditFlags            : None
```

From the result, we can see that one GPO allows all Domain Users full write access. We can then confirm the name of the GPO using the built-in cmdlet `Get-GPO`.

```powershell-session
PS C:\htb> Get-GPO -Guid 831DE3ED-40B1-4703-ABA7-8EA13B2EB118

DisplayName      : Screensaver
DomainName       : INLANEFREIGHT.LOCAL
Owner            : INLANEFREIGHT\Domain Admins
Id               : 831de3ed-40b1-4703-aba7-8ea13b2eb118
GpoStatus        : AllSettingsEnabled
Description      :
CreationTime     : 8/26/2020 10:46:46 PM
ModificationTime : 8/26/2020 11:11:01 PM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 0, SysVol Version: 0
WmiFilter        :
```

This misconfigured GPO could be exploited using a tool such as `SharpGPOAbuse` and the `--AddUserRights` attack to give a user unintended rights or the `--AddLocalAdmin` attack to add a user as a local admin on a machine where the GPO is applied and use it to move laterally towards our target.

---

## Hidden GPO Code Execution Paths
Group Policy is the most basic way System Administrators can command many Computers to perform a task. It is not the most common way to do things as many organizations will use commercial applications such as:

- Microsoft SCCM - System Center Configuration Manager
- PDQInventory/Deploy
- NinjaRMM (Remote Management and Monitoring)
- Ansible/Puppet/Salt

However, each one of these applications is non-default, and when an Administrator googles for a solution, their answer probably won't include the technology they use. Often, you may find one-off configurations an administrator did to accomplish a task quickly. For example, on multiple occasions, I have run across a "Machine/User Startup" script to collect inventory and write it to a domain share. I have seen this policy execute both BAT and VBScript files that were either write-able by the `machine account` or `domain users`. Whenever I dig into file shares and see files write-able by Everyone, Authenticated Users, Domain Users, Domain Computers, etc., containing what looks like log files, I dig into Group Policy, specifically looking for Startup Scripts.

That is just one way an Administrators use "Code Execution via GP" legitimately. Here is a list of the path's I know about:

- Add Registry Autoruns
- Software Installation (Install MSI Package that exists on a share)
- Scripts in the Startup/Shutdown for a Machine or User
- Create Shortcuts on Desktops that point to files
- Scheduled Tasks

If anyone of these paths points to a file on a share, enumerate the permissions to check if non-administrators can edit the file. Your tools will often miss this because it only looks at if the Group Policy itself is write-able, not if the executables/scripts the group policy references are writeable.

---
