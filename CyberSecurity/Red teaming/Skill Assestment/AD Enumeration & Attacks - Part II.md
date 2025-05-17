
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

-----

## Foothold

I need to view how can access in this client we the credential founded so i enumerate the open port of **172.16.7.3**
```bash
sudo nmap -sC -sV -oA nmap/nmap_scan 172.16.7.3
```

![[Pasted image 20250517151754.png]]

after scan i have the NetBIOS name `DC01` and the `389` port is open this means is possible to access with RDP protocol

I have also discover a 5985 port open this meas the `winrm` is enable
```bash
sudo nmap -p 5985,5986 172.16.7.3
```
![[Pasted image 20250517151931.png]]
