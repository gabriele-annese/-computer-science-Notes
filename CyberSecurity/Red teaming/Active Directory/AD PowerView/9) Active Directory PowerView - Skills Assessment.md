

*What is the passwordhistorysize of the domain?*

```bash
Get-DomainPolicy
```

![[Pasted image 20250630230810.png]]

*Answer: 24*

----

*What is the SID of the user rachel.flemmings?*

```bash
Get-ADUser -Identity "rachel.flemmings" -Properties * | Select name,objectSid
```

![[Pasted image 20250630231210.png]]
*Answer: S-1-5-21-3394586996-1871716043-2583881113-1105*

----

*What is the domain functional level? (1 single number)*

```bash
Get-Domain | select DomainModeLevel
```

![[Pasted image 20250630231559.png]]

*Answer: 5*

----

*What GPO is applied to the ENUM2-MS01 host? (case sensitive)*

```bash
Get-DomainGPO -ComputerName "ENUM2-DC01" | select displayname
```
![[Pasted image 20250630231856.png]]

*Answer: Disable Defender*

-----
*Find a non-standard share on the ENUM2-DC01 host. Access it and submit the contents of share.txt.*

```bash
Get-NetShare -ComputerName "ENUM2-DC01"
```

![[Pasted image 20250630232242.png]]

*Answer: HTB{r3v1ew_s4ar3_p3Rms!}*

----
*Find a domain computer with a password in the description field. Submit the password as your answer.*

```bash
Get-DomainComputer -Properties * | Where-Object {$_.description -ne $null} | select name, description, objectsid
```
![[Pasted image 20250630233125.png]]

*Answer: Just_f0r_adm1n_@cess!*

----

*Who is the group manager of the Citrix Admins group?*

```bash
Get-DomainManagedSecurityGroup | Where-Object {$_.GroupName -eq "Citrix Admins"}
```

![[Pasted image 20250630233629.png]]

*Answer: poppy.louis*

