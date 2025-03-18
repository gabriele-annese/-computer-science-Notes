## Task 1: Manage Users

Our first task of the day includes adding a few new-hire users into AD. We are just going to create them under the `"inlanefreight.local"` scope, drilling down into the `"Corp > Employees > HQ-NYC > IT "` folder structure for now. Once we create our other groups, we will move them into the new folders. You can utilize the Active Directory PowerShell module (New-ADUser), the Active Directory Users and Computers snap-in, or MMC to perform these actions.

#### Users to Add:

|**User**|
|---|
|`Andromeda Cepheus`|
|`Orion Starchaser`|
|`Artemis Callisto`|

Each user should have the following attributes set, along with their name:

|**Attribute**|
|---|
|`full name`|
|`email (first-initial.lastname@inlanefreight.local) ( ex. j.smith@inlanefreight.local )`|
|`display name`|
|`User must change password at next logon`|

Once we have added our new hires, take a quick second and remove a few old user accounts found in an audit that are no longer required.

### Task 1: Solution

1. Create a new OU with "NewHire" name
	![[Pasted image 20250317224621.png]]

2. Create a new hire users. Example of `Andromeda Cepheus`
   ![[Pasted image 20250317224713.png]]
   
   click `next` and check the `User must change password at next logon`.
   ![[Pasted image 20250317224942.png]]
   
   ![[Pasted image 20250317225002.png]]
   
   ![[Pasted image 20250317225004.png]]
   
3. Now try to create the new user with `New-ADUser` PowerShell's module
   i use this powershell script
```powershell
$splat = @{
    Name = 'Orion Starchaser'
    GivenName = 'Orion'
    Surname = 'Starchaser'
    EmailAddress = 'o.starchaser@inlanefreight.local'
    DisplayName = 'Orion Starchaser'
    UserPrincipalName = 'o.starchaser@inlanefreight.local'
    SamAccountName = 'o.starchaser'
    Path = 'OU=NewHire,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL'
    AccountPassword = (Read-Host -AsSecureString 'AccountPassword')
    Enabled = $true
    ChangePasswordAtLogon = $true
}
New-ADUser @splat
```

All properties are documented in [microsoft documentation ](https://learn.microsoft.com/en-us/powershell/module/activedirectory/new-aduser?view=windowsserver2025-ps).
	![[Pasted image 20250317232733.png]]
	same attributes
	![[Pasted image 20250317232739.png]]

3. Remove user from AD. First i use this query to find `ObjectGUID` with `Get-ADUser` module
```powershell
	Get-AdUser -Filter {GivenName -eq 'Mike' -and UserPrincipalName -eq 'mohare@INLANEFREIGHT.LOCAL'}
```

![[Pasted image 20250317234305.png]]
	to view the user under the `AD User and Computer` you need to follow the `DistinguishedName`. In this case is under `Employees > Financial-LON > Audit`
	![[Pasted image 20250317233816.png]] 
	now using `Remove-ADUser` module i delete this user

```powershell
	Remove-ADUser -Identity "CN=Mike O'Hare,OU=Audit,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL" -Confirm:$false
```

4. Unlock the Adam Master's account. First with `Get-ADUser` module filter and find information by the user. In this case the property that we need to view is the `LockedOut` if it's set to true the user is locked.
```powershell
Get-AdUser -Filter {GivenName -eq 'Adam' -and UserPrincipalName -eq 'Amasters@INLANEFREIGHT.LOCAL'} -Properties LockedOut
```

To unlock the user we can utilize the `Unlock-ADAccount` module
```powershell
	Unlock-ADAccount -Identity "CN=Adam Masters,OU=Interns,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
```

from GUI once founded the user select `Reset password` option
	![[Pasted image 20250317235824.png]]

## Task 2: Manage Groups and Other Organizational Units

Next up for us is to create a new Security Group called `Security Analysts` and then add our new hires into the group. This group should also be nested in an OU named the same under the `IT` hive. The `New-ADOrganizationalUnit` PowerShell command should enable you to quickly add a new security group. We can also utilize the AD Users and Computers snap-in like in Task-1 to complete this task.

### Task 2: Solution

1. Create a OU using `New-ADOrganizationalUnit` module
```powershell
New-ADOrganizationalUnit -Name "Security Analysts" -Path "OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
```

![[Pasted image 20250318000826.png]]

2. Create a Security group under new OU with `New-ADGroup` module
```powershell
New-ADGroup -Name "Security Analysts" -SamAccountName analysts -GroupCategory Security -GroupScope Global -DisplayName "Security Analysts" -Path "OU=Security Analysts,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL" -Description "Members of this group are Security Analysts under the IT OU"
```
![[Pasted image 20250318001737.png]]

2. Now add new hire user to `Sercurity Analysts` groups with `Add-ADGroupMember` module.
```powershell
Add-ADGroupMember -Identity analysts -Members "a.cepheus","o.starchaser"
```
![[Pasted image 20250318003349.png]]

## Task 3: Manage Group Policy Objects

Next, we have been asked to duplicate the group policy `Logon Banner`, rename it `Security Analysts Control`, and modify it to work for the new Analysts OU. We will need to make the following changes to the Policy Object:

- we will be modifying the Password policy settings for users in this group and expressly allowing users to access PowerShell and CMD since their daily duties require it.
- For computer settings, we need to ensure the Logon Banner is applied and that removable media is blocked from access.

Once done, make sure the Group Policy is applied to the `Security Analysts` OU. This will require the use of the Group Policy Management snap-in found under `Tools` in the Server Manager window. For more of a challenge, the `Copy-GPO` cmdlet in PowerShell can also be utilized.

### Task 3: Solution

1. Duplicate the Object via PowerShell
```powerhsell
Copy-GPO -SourceName "Logon Banner" -TargetName "Security Analysts Control"
```
2. Link the New GPO to an OU
```powershell
New-GPLink -Name "Security Analysts Control" -Target "ou=Security Analysts,ou=IT,OU=HQ-NYC,OU=Employees,OU=Corp,dc=INLANEFREIGHT,dc=LOCAL" -LinkEnabled Yes

```

3. Now for modify the gpo it necessary open the GPMC.
	To disable the command prompt click `Edit` on `Prevenet accstot he command prompt` policy and disable
   ![[Pasted image 20250318004816.png]]
   ![[Pasted image 20250318004951.png]]
   
4. Disable all removable storage. Edit the own gpo
   ![[Pasted image 20250318005339.png]]
   under `user Configuration > System > Removable Storage Access` and enable the policy `All Removable Storgae classes: Deny all access`
   ![[Pasted image 20250318005508.png]]  ![[Pasted image 20250318005645.png]]
   ![[Pasted image 20250318005647.png]]
   
   5. Computer Configuration Group Policies
      ![[Pasted image 20250318010044.png]]
      Check if are enalble
      ![[Pasted image 20250318010211.png]]
      passoword policies
      ![[Pasted image 20250318010302.png]]
      set to 10 the `minimum passoword length` policy
      ![[Pasted image 20250318010325.png]]
    
    enable the `Password must meet complexity requirements `policy
    ![[Pasted image 20250318010507.png]]
    
    set to 5 the `Enforce password history` policy
    ![[Pasted image 20250318010709.png]]
    
    Set the `Minimum Password Age` setting by defining the setting and applying a minimum age of 7 days. A new window will pop up telling us that the setting for "Maximum Password Age" will be set as well.
    ![[Pasted image 20250318010741.png]]
    
    Validate all the settings match what we wished to define. If all looks well, we have completed this task!
    ![[Pasted image 20250318010840.png]]

    