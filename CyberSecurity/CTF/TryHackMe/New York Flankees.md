## Recon
Fast record to check all open ports
```shell
sudo nmap -sS -p- -T5 10.10.29.193                                                               
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-26 16:43 EDT                                   
Nmap scan report for 10.10.29.193 (10.10.29.193)                                                     
Host is up (0.039s latency).                                                                         
Not shown: 65533 closed tcp ports (reset)                                                            
PORT     STATE SERVICE                                                                               
22/tcp   open  ssh       
8080/tcp open  http-proxy                         

Nmap done: 1 IP address (1 host up) scanned in 24.74 seconds 
```

ok now retrieve more information about this two port 
```shell
sudo nmap -g 53 -sV -sC -p 22,8080 -T5 10.10.29.193                                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-26 16:44 EDT                                   
Nmap scan report for 10.10.29.193 (10.10.29.193)                                                     
Host is up (0.038s latency).                      

PORT     STATE SERVICE VERSION                    
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)                 
| ssh-hostkey:           
|   3072 53:13:a1:ad:a4:97:3f:86:ff:a9:18:b4:c7:d3:49:73 (RSA)                                       
|   256 0a:11:29:f2:8c:ce:4e:32:00:1a:6a:d1:eb:f5:29:ba (ECDSA)                                      
|_  256 3e:d8:98:e8:55:ac:37:70:13:52:25:7e:57:45:c2:39 (ED25519)                                    
8080/tcp open  http    Octoshape P2P streaming web service                                           
|_http-title: Hello world!                        
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                              

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .       
Nmap done: 1 IP address (1 host up) scanned in 8.99 seconds  
```

in the `debug.html` source page i found this function
```js
function stefanTest1002() {
        var xhr = new XMLHttpRequest();
        var url = "http://localhost/api/debug";
        // Submit the AES/CBC/PKCS payload to get an auth token
        // TODO: Finish logic to return token
        xhr.open("GET", url + "/39353661353931393932373334633638EA0DCC6E567F96414433DDF5DC29CDD5E418961C0504891F0DED96BA57BE8FCFF2642D7637186446142B2C95BCDEDCCB6D8D29BE4427F26D6C1B48471F810EF4", true);

        xhr.onreadystatechange = function () {
            if (xhr.readyState === 4 && xhr.status === 200) {
                console.log("Response: ", xhr.responseText);
            } else {
                console.error("Failed to send request.");
            }
        };
        xhr.send();
    }
```

![[Pasted image 20240827001636.png]]
infect in network inspector i see the call 
![[Pasted image 20240827001746.png]]
if i try to call receive the '**Custom authentication success**' message
![[Pasted image 20240827002148.png]]
## Exploit

## Loot
