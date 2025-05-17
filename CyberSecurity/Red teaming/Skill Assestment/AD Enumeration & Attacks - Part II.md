
## Scenario

Our client Inlanefreight has contracted us again to perform a full-scope internal penetration test. The client is looking to find and remediate as many flaws as possible before going through a merger & acquisition process. The new CISO is particularly worried about more nuanced AD security flaws that may have gone unnoticed during previous penetration tests. The client is not concerned about stealth/evasive tactics and has also provided us with a Parrot Linux VM within the internal network to get the best possible coverage of all angles of the network and the Active Directory environment. Connect to the internal attack host via SSH (you can also connect to it using `xfreerdp` as shown in the beginning of this module) and begin looking for a foothold into the domain. Once you have a foothold, enumerate the domain and look for flaws that can be utilized to move laterally, escalate privileges, and achieve domain compromise.

----

To obtain a **NTLM** hash i used the **responder** tool
```bash
responder -I ens24 -wrf
```

- `-I`: indicate which network interface we'll be use
- `-w`: Start the WPAD rogue proxy server. Default value is Upstream HTTP proxy used by the rogue WPAD Proxy for outgoing requests (format: host:port)
- `-r`: Enable answers for netbios wredir suffix queries. Answering to wredir will likely break stuff on the network. Default: False
