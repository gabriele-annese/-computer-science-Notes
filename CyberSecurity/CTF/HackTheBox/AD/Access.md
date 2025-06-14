# Machine info
Access is an easy difficulty machine, that highlights how machines associated with the physical security of an environment may not themselves be secure. Also highlighted is how accessible FTP/file shares can often lead to getting a foothold or lateral movement. It teaches techniques for identifying and exploiting saved credentials.
# Enumeration

## Nmap
```bash
nmap -sC -sV -p- -oA nmap/scan 10.129.237.193
Nmap scan report for 10.129.237.193
Host is up (0.0075s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jun 14 07:03:01 2025 -- 1 IP address (1 host up) scanned in 322.17 seconds
```

We have tree tcp open ports. using `-sC` flag we can notice that in the `21` port the **Anonymous auth** is allow.

## FTP
using `ftp` tool we can access to the machine with `Anonymous` user and blank password
![[Pasted image 20250614194155.png]]

- `passive`: When we use an `active mode` the clients opens a port and the server connects back to the client, instance in the `passive mode` the server opens a random port and tells the client to connect top that port. **The passive mode is great to use when a client is a behind a firewall or NAT**

browsing through folder we can found the `backup.mdb` file.

>NOTE
>
>What is a `mdb` file?
>
>A **Microsoft Access Database** file format is used to store data in a structured way.MDB files contain database queries, tables, and more that can be used to link to and store data from other files, like [XML](https://www.lifewire.com/what-is-an-xml-file-2622560) and [HTML](https://www.lifewire.com/htm-html-file-2621691), and applications, like [Excel](https://www.lifewire.com/what-is-microsoft-excel-3573533) and [SharePoint](https://www.lifewire.com/what-is-sharepoint-4176266). Its possible to query this file using TSQL

Perfect now that we now what this file is how we can get in our attack machine? using `get` command in the ftp tool? Hehehehe not to easy my little hacker 

ftp has two transfers modes

|Mode|Use Case|Problem|
|---|---|---|
|`ASCII`|Text files (e.g., `.txt`, `.html`)|Converts line endings (CR/LF) — corrupts binary files like `.mdb`|
|`Binary`|All binary files (e.g., `.mdb`, `.zip`, `.jpg`)|Transfers bytes exactly as-is|

in this case first to get we need to type `binary` to switch the transfer mode
![[Pasted image 20250614195913.png]]


or u can use the `wget` command with the flag `--no-passive`

```bash
wget --no-passive-ftp ftp://Anonymous@10.129.237.193/Backups/backup.mdb
```

## Enumerate MDB file
TODO
write bash script to extract all of data for backup.mdb



# Foothold
# Privilege Escalation
