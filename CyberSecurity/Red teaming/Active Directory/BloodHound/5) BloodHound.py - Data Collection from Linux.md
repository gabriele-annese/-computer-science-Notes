[SharpHound](https://github.com/BloodHoundAD/SharpHound) does not have an official data collector tool for Linux. However, [Dirk-jan Mollema](https://twitter.com/_dirkjan) created [BloodHound.py](https://github.com/fox-it/BloodHound.py), a Python-based collector for BloodHound based on Impacket, to allow us to collect Active Directory information from Linux for BloodHound.

---

## Installation

We can install [BloodHound.py](https://github.com/fox-it/BloodHound.py) with `pip install bloodhound` or by cloning its repository and running `python setup.py install`. To run, it requires `impacket`, `ldap3`, and `dnspython`. The tool can be installed via pip by typing the following command:

#### Install BloodHound.py

  BloodHound.py - Data Collection from Linux

```shell-session
BusySec@htb[/htb]$ pip install bloodhound
Defaulting to user installation because normal site-packages is not writeable
Collecting bloodhound
  Downloading bloodhound-1.6.0-py3-none-any.whl (80 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 81.0/81.0 kB 1.8 MB/s eta 0:00:00
Requirement already satisfied: ldap3!=2.5.0,!=2.5.2,!=2.6,>=2.5 in /home/plaintext/.local/lib/python3.9/site-packages (from bloodhound) (2.9.1)
Requirement already satisfied: pyasn1>=0.4 in /usr/local/lib/python3.9/dist-packages (from bloodhound) (0.4.6)
Requirement already satisfied: impacket>=0.9.17 in /home/plaintext/.local/lib/python3.9/site-packages (from bloodhound) (0.10.1.dev1+20220720.103933.3c6713e3)
Requirement already satisfied: future in /usr/lib/python3/dist-packages (from bloodhound) (0.18.2)
Requirement already satisfied: dnspython in /usr/local/lib/python3.9/dist-packages (from bloodhound) (2.2.1)
Requirement already satisfied: charset-normalizer in /home/plaintext/.local/lib/python3.9/site-packages (from impacket>=0.9.17->bloodhound) (2.0.12)
Requirement already satisfied: six in /usr/local/lib/python3.9/dist-packages (from impacket>=0.9.17->bloodhound) (1.12.0)
Requirement already satisfied: pycryptodomex in /usr/lib/python3/dist-packages (from impacket>=0.9.17->bloodhound) (3.9.7)
Requirement already satisfied: ldapdomaindump>=0.9.0 in /usr/lib/python3/dist-packages (from impacket>=0.9.17->bloodhound) (0.9.3)
Requirement already satisfied: dsinternals in /home/plaintext/.local/lib/python3.9/site-packages (from impacket>=0.9.17->bloodhound) (1.2.4)
Requirement already satisfied: flask>=1.0 in /usr/lib/python3/dist-packages (from impacket>=0.9.17->bloodhound) (1.1.2)
Requirement already satisfied: pyOpenSSL>=21.0.0 in /home/plaintext/.local/lib/python3.9/site-packages (from impacket>=0.9.17->bloodhound) (22.1.0)
Requirement already satisfied: cryptography<39,>=38.0.0 in /home/plaintext/.local/lib/python3.9/site-packages (from pyOpenSSL>=21.0.0->impacket>=0.9.17->bloodhound) (38.0.3)
Requirement already satisfied: cffi>=1.12 in /usr/local/lib/python3.9/dist-packages (from cryptography<39,>=38.0.0->pyOpenSSL>=21.0.0->impacket>=0.9.17->bloodhound) (1.12.3)
Requirement already satisfied: pycparser in /usr/local/lib/python3.9/dist-packages (from cffi>=1.12->cryptography<39,>=38.0.0->pyOpenSSL>=21.0.0->impacket>=0.9.17->bloodhound) (2.19)
Installing collected packages: bloodhound
Successfully installed bloodhound-1.6.0
```

To install it from the source, we can clone the [BloodHound.py GitHub repository](https://github.com/fox-it/BloodHound.py) and run the following command:

#### Install BloodHound.py from the source

  BloodHound.py - Data Collection from Linux

```shell-session
BusySec@htb[/htb]$ git clone https://github.com/fox-it/BloodHound.py -q
BusySec@htb[/htb]$ cd BloodHound.py/
BusySec@htb[/htb]$ sudo python3 setup.py install
running install
running bdist_egg
running egg_info
creating bloodhound.egg-info
writing bloodhound.egg-info/PKG-INFO
writing dependency_links to bloodhound.egg-info/dependency_links.txt
writing entry points to bloodhound.egg-info/entry_points.txt
writing requirements to bloodhound.egg-info/requires.txt
writing top-level names to bloodhound.egg-info/top_level.txt
writing manifest file 'bloodhound.egg-info/SOURCES.txt'
reading manifest file 'bloodhound.egg-info/SOURCES.txt'
writing manifest file 'bloodhound.egg-info/SOURCES.txt'
installing library code to build/bdist.linux-x86_64/egg
running install_lib
running build_py
creating build
creating build/lib
creating build/lib/bloodhound
copying bloodhound/__init__.py -> build/lib/bloodhound
copying bloodhound/__main__.py -> build/lib/bloodhound
...SNIP...
```

## BloodHound.py Options

We can use `--help` to list all `BloodHound.py` options. The following list corresponds to version 1.6.0:

#### BloodHound.py options

  BloodHound.py - Data Collection from Linux

```shell-session
BusySec@htb[/htb]$ python3 bloodhound.py 
usage: bloodhound.py [-h] [-c COLLECTIONMETHOD] [-d DOMAIN] [-v] [-u USERNAME] [-p PASSWORD] [-k] [--hashes HASHES] [-no-pass] [-aesKey hex key]
                     [--auth-method {auto,ntlm,kerberos}] [-ns NAMESERVER] [--dns-tcp] [--dns-timeout DNS_TIMEOUT] [-dc HOST] [-gc HOST]
                     [-w WORKERS] [--exclude-dcs] [--disable-pooling] [--disable-autogc] [--zip] [--computerfile COMPUTERFILE]
                     [--cachefile CACHEFILE]

Python based ingestor for BloodHound
For help or reporting issues, visit https://github.com/Fox-IT/BloodHound.py

optional arguments:
  -h, --help            show this help message and exit
  -c COLLECTIONMETHOD, --collectionmethod COLLECTIONMETHOD
                        Which information to collect. Supported: Group, LocalAdmin, Session, Trusts, Default (all previous), DCOnly (no computer
                        connections), DCOM, RDP,PSRemote, LoggedOn, Container, ObjectProps, ACL, All (all except LoggedOn). You can specify more
                        than one by separating them with a comma. (default: Default)
  -d DOMAIN, --domain DOMAIN
                        Domain to query.
  -v                    Enable verbose output

authentication options:
  Specify one or more authentication options. 
  By default Kerberos authentication is used and NTLM is used as fallback. 
  Kerberos tickets are automatically requested if a password or hashes are specified.

  -u USERNAME, --username USERNAME
                        Username. Format: username[@domain]; If the domain is unspecified, the current domain is used.
  -p PASSWORD, --password PASSWORD
                        Password
  -k, --kerberos        Use kerberos
  --hashes HASHES       LM:NLTM hashes
  -no-pass              don't ask for password (useful for -k)
  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)
  --auth-method {auto,ntlm,kerberos}
                        Authentication methods. Force Kerberos or NTLM only or use auto for Kerberos with NTLM fallback

collection options:
  -ns NAMESERVER, --nameserver NAMESERVER
                        Alternative name server to use for queries
  --dns-tcp             Use TCP instead of UDP for DNS queries
  --dns-timeout DNS_TIMEOUT
                        DNS query timeout in seconds (default: 3)
  -dc HOST, --domain-controller HOST
                        Override which DC to query (hostname)
  -gc HOST, --global-catalog HOST
                        Override which GC to query (hostname)
  -w WORKERS, --workers WORKERS
                        Number of workers for computer enumeration (default: 10)
  --exclude-dcs         Skip DCs during computer enumeration
  --disable-pooling     Don't use subprocesses for ACL parsing (only for debugging purposes)
  --disable-autogc      Don't automatically select a Global Catalog (use only if it gives errors)
  --zip                 Compress the JSON output files into a zip archive
  --computerfile COMPUTERFILE
                        File containing computer FQDNs to use as allowlist for any computer based methods
  --cachefile CACHEFILE
                        Cache file (experimental)
```

As BloodHound.py uses `impacket`, options such as the use of hashes, ccachefiles, aeskeys, kerberos, among others, are also available for BloodHound.py.

---

## Using BloodHound.py

To use BloodHound.py in Linux, we will need `--domain` and `--collectionmethod` options and the authentication method. Authentication can be a username and password, an NTLM hash, an AES key, or a ccache file. BloodHound.py will try to use the Kerberos authentication method by default, and if it fails, it will fall back to NTLM.

Another critical piece is the domain name resolution. If our DNS server is not the domain DNS server, we can use the option `--nameserver`, which allows us to specify an alternative name server for queries.

#### Running BloodHound.py

  BloodHound.py - Data Collection from Linux

```shell-session
BusySec@htb[/htb]$ bloodhound-python -d inlanefreight.htb -c DCOnly -u htb-student -p HTBRocks! -ns 10.129.204.207 -k                                                                    
INFO: Found AD domain: inlanefreight.htb                                                                                                                                   
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (inlanefreight.htb:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: dc01.inlanefreight.htb                 
INFO: Found 1 domains                                                                                                                                                      
INFO: Found 1 domains in the forest                                                  
INFO: Connecting to LDAP server: dc01.inlanefreight.htb                                                                                                                    
INFO: Found 6 users                  
INFO: Found 52 groups                                                                                                                                                      
INFO: Found 2 gpos                                                                   
INFO: Found 1 ous                                                                                                                                                          
INFO: Found 19 containers                                                            
INFO: Found 3 computers                                                              
INFO: Found 0 trusts                                                                 
INFO: Done in 00M 11S
```

**Note:** Kerberos authentication requires the host to resolve the domain FQDN. This means that the option `--nameserver` is not enough for Kerberos authentication because our host needs to resolve the DNS name KDC for Kerberos to work. If we want to use Kerberos authentication, we need to set the DNS Server to the target machine or configure the DNS entry in our hosts' file. Due to the requirements of Kerberos we must synchronize the time between the KDC and the attacker host using `sudo ntpdate [TARGET_IP]`.

Let's add the DNS entry in our hosts file:

#### Setting up the /etc/hosts file

  BloodHound.py - Data Collection from Linux

```shell-session
BusySec@htb[/htb]$ echo -e "\n10.129.204.207 dc01.inlanefreight.htb dc01 inlanefreight inlanefreight.htb" | sudo tee -a /etc/hosts

10.129.204.207 dc01.inlanefreight.htb dc01 inlanefreight inlanefreight.htb
```

Use BloodHound.py with Kerberos authentication:

#### Using BloodHound.py with Kerberos authentication

  BloodHound.py - Data Collection from Linux

```shell-session
BusySec@htb[/htb]$ bloodhound-python -d inlanefreight.htb -c DCOnly -u htb-student -p HTBRocks! -ns 10.129.204.207 --kerberos
INFO: Found AD domain: inlanefreight.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.inlanefreight.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Connecting to LDAP server: dc01.inlanefreight.htb
INFO: Found 6 users
INFO: Found 52 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 3 computers
INFO: Found 0 trusts
INFO: Done in 00M 11S
```

**Note:** Kerberos Authentication is the default authentication method for Windows. Using this method instead of NTLM makes our traffic look more normal.

## BloodHound.py Output files

Once the collection finishes, it will produce the JSON files, but by default, it won't zip the content as SharpHound does. If we want the content to be placed in a zip file, we need to use the option `--zip`.

#### List All Collections of BloodHound.py

  BloodHound.py - Data Collection from Linux

```shell-session
BusySec@htb[/htb]$ ls
20230112171634_computers.json   20230112171634_domains.json  20230112171634_groups.json  20230112171634_users.json
20230112171634_containers.json  20230112171634_gpos.json     20230112171634_ous.json
```

