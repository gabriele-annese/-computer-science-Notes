## RSAT Background

The `Remote Server Administration Tools` (`RSAT`) have been part of Windows since the days of Windows 2000. RSAT allows systems administrators to remotely manage Windows Server roles and features from a workstation running Windows 10, Windows 8.1, Windows 7, or Windows Vista. `RSAT` can only be installed on Professional or Enterprise editions of Windows. In an enterprise environment, RSAT can remotely manage Active Directory, DNS, and DHCP. RSAT also allows us to manage installed server roles and features, File Services, and Hyper-V. The full listing of tools included with `RSAT` is:

- SMTP Server Tools
- Hyper-V Management Tools
- Hyper-V Module for Windows PowerShell
- Hyper-V GUI Management Tools
- Windows Server Update Services Tools
- API and PowerShell cmdlets
- User Interface Management Console
- Active Directory Users and Computers Snap-in
- Active Directory Sites and Services Snap-in
- Active Directory Domains and Trusts Snap-in
- Active Directory Administrative Center Snap-in
- ADSI Edit Snap-in
- Active Directory Schema Snap-in (Not Registered)
- Active Directory Command Line Tools
- Active Directory Module for Windows PowerShell
- IIS Management Tools
- IIS Management Console
- IIS Management Compatibility
- Feature Tools
- Remote Desktop Services Tools
- Role Tools
- Update Services Tools
- Group Policy Tools

This [script](https://gist.github.com/dually8/558fcfa9156f59504ab36615dfc4856a) can be used to install RSAT in Windows 10 1809, 1903, and 1909. Installation instructions for other versions of Windows, as well as additional information about RSAT, can be found [here](https://support.microsoft.com/en-us/help/2693643/remote-server-administration-tools-rsat-for-windows-operating-systems). RSAT can be installed easily with PowerShell as well.

We can check which, if any, RSAT tools are installed using PowerShell.

#### PowerShell - Available RSAT Tools
```powershell
PS C:\htb>  Get-WindowsCapability -Name RSAT* -Online | Select-Object -Property Name, State

Name                                                          State
----                                                          -----
Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0             NotPresent
Rsat.BitLocker.Recovery.Tools~~~~0.0.1.0                 NotPresent
Rsat.CertificateServices.Tools~~~~0.0.1.0                NotPresent
Rsat.DHCP.Tools~~~~0.0.1.0                               NotPresent
Rsat.Dns.Tools~~~~0.0.1.0                                NotPresent
Rsat.FailoverCluster.Management.Tools~~~~0.0.1.0         NotPresent
Rsat.FileServices.Tools~~~~0.0.1.0                       NotPresent
Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0             NotPresent
Rsat.IPAM.Client.Tools~~~~0.0.1.0                        NotPresent
Rsat.LLDP.Tools~~~~0.0.1.0                               NotPresent
Rsat.NetworkController.Tools~~~~0.0.1.0                  NotPresent
Rsat.NetworkLoadBalancing.Tools~~~~0.0.1.0               NotPresent
Rsat.RemoteAccess.Management.Tools~~~~0.0.1.0            NotPresent
Rsat.RemoteDesktop.Services.Tools~~~~0.0.1.0             NotPresent
Rsat.ServerManager.Tools~~~~0.0.1.0                      NotPresent
Rsat.Shielded.VM.Tools~~~~0.0.1.0                        NotPresent
Rsat.StorageMigrationService.Management.Tools~~~~0.0.1.0 NotPresent
Rsat.StorageReplica.Tools~~~~0.0.1.0                     NotPresent
Rsat.SystemInsights.Management.Tools~~~~0.0.1.0          NotPresent
Rsat.VolumeActivation.Tools~~~~0.0.1.0                   NotPresent
Rsat.WSUS.Tools~~~~0.0.1.0                               NotPresent
```

From here, we can choose to install all available tools using the following command:
#### PowerShell - Install All Available RSAT Tools

```powershell
PS C:\htb> Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability –Online
```

We can also install tools one at a time as needed.

#### PowerShell - Install an RSAT Tool

```powershell-session
PS C:\htb> Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0  –Online
```

Once installed, all of the tools will be available under `Administrative Tools` in the `Control Panel`.

![[Pasted image 20250604001221.png]]

-----
## Domain Context for Enumeration

Many tools are missing credential and context parameters and instead get those values directly from the current context. There are a few ways to alter a user's context in Windows if you have access to a password or a hash, such as:

Using "`runas /netonly`" to leverage the built-in [runas.exe](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525\(v=ws.11\)) command line tool.

#### CMD - Runas User

```cmd
C:\htb> runas /netonly /user:htb.local\jackie.may powershell
```

Other tools that we will discuss in later modules, such as [Rubeus](https://github.com/GhostPack/Rubeus) and [mimikatz](https://github.com/gentilkiwi/mimikatz) can be passed cleartext credentials or an NTLM password hash.

#### CMD - Rubeus.exe Cleartext Credentials

```cmd
C:\htb> rubeus.exe asktgt /user:jackie.may /domain:htb.local /dc:10.10.110.100 /rc4:ad11e823e1638def97afa7cb08156a94
```

#### CMD - Mimikatz.exe Cleartext Credentials

```cmd
C:\htb> mimikatz.exe sekurlsa::pth /domain:htb.local /user:jackie.may /rc4:ad11e823e1638def97afa7cb08156a94
```

---

## Enumeration with RSAT

If we compromise a domain-joined system (or a client has you perform an AD assessment from one of their workstations), we can leverage RSAT to enumerate AD. While RSAT will make GUI tools such as `Active Directory Users and Computers` and `ADSI Edit` available to us, the most important tool we have seen throughout this module is the PowerShell [Active Directory module](https://github.com/MicrosoftDocs/windows-powershell-docs/blob/main/docset/winserver2012-ps/adcsadministration/adcsadministration.md).

Alternatively, we can enumerate the domain from a non-domain joined host (provided that it is in a subnet that communicates with a domain controller) by launching any RSAT snap-ins using "`runas`" from the command line. This is particularly useful if we find ourselves performing an internal assessment, gain valid AD credentials, and would like to perform enumeration from a Windows VM.

![image](https://academy.hackthebox.com/storage/modules/22/rsat_adsi.png)

---

We can also open the `MMC Console` from a non-domain joined computer using the following command syntax:

#### CMD - MMC Runas Domain User

```cmd
C:\htb> runas /netonly /user:Domain_Name\Domain_USER mmc
```

![image](https://academy.hackthebox.com/storage/modules/22/mmc_non_ad.png)

We can add any of the RSAT snap-ins and enumerate the target domain in the context of the target user `sally.jones` in the `freightlogistics.local` domain. After adding the snap-ins, we will get an error message that the "specified domain either does not exist or could not be contacted." From here, we have to right-click on the `Active Directory Users and Computers` snap-in (or any other chosen snap-in) and choose `Change Domain`.

![image](https://academy.hackthebox.com/storage/modules/22/mmc_change_domain.png)

Type the target domain into the `Change domain` dialogue box, here `freightlogistics.local`. From here, we can now freely enumerate the domain using any of the AD RSAT snapins.

![image](https://academy.hackthebox.com/storage/modules/22/mmc_ad_users_computers.png)

While these graphical tools are useful and easy to use, they are very inefficient when trying to enumerate a large domain. In the next few sections, we will introduce `LDAP` and various types of search filters that we can use to enumerate AD using PowerShell. The topics that we cover in these sections will help us gain a better understanding of how AD works and how to search for information efficiently, which will ultimately better inform us on the usage of the more "automated" tools and scripts that we will cover in the next two `AD Enumeration` modules.
