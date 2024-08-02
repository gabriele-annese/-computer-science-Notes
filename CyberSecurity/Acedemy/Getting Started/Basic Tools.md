---
cssclasses:
  - recolor-images
tags:
  - vim
  - tmux
  - netcat
  - socat
  - ssh
---
## SSH
**Secure Shell (SSH)** is a network protocol run on port **22** by default. SSH provide a secure way to user/administrator to access the machine. SSH can be configure with password or with **SSH public/private key pair**. 

SSH use a client-server model, connection a user running an SSH client application such as **OpenSSH** to an SSH server.
SSH connection is typically more efficient respect to a revers shell and can often be used as a "jump host" to enumerate and attack other hosts in the network or setup a persistence.

## Netcat
**Netcat (NC)** is a network utility for intefacing with TCP/UDP ports. It's often use like a listener for a reverse shell. But using netcat is possible to connect any listening port.

>NOTE
>If the port we try to connect is occupied by another service nc return a **Banner** of the service running in that port. This technique is called **Banner Grabbing**.

```shell

BusySec@htb[/htb]$ netcat 10.10.10.10 22
SSH-2.0-OpenSSH_8.4p1 Debian-3

```

in Linux is preinstalled and it's possible download it in Windows. In windows exist an alternative called **PowerCat** running on powershell.

## Socat
Socat is a powerful Netcat tool. Socat can forwarding ports and can manage different device. Whit socat it's possible upgrade the shell to obtain a **shell to a fully interface (TTY)**. Socat is a simple binary this means is transportable in a infected machine to have better connection.

## Tmux
Tmux is a terminal multiplexers. Some feature of tmux

**Vertical split view**
![[Pasted image 20240803012530.png]]

**Horizontal split view**
![[Pasted image 20240803012558.png]]
This [cheatsheet](https://tmuxcheatsheet.com/) is a very handy reference.

## Vim
Vim is a great text editor can be used for writing code or editing text files on Linux system.

This [cheatsheet](https://vimsheet.com/) is an excellent resource for further unlocking the power of `Vim`.
