Default webpage for nginx web server:

![](attachments/Pasted%20image%2020251127112555.png)

We also find the version of the `nginx`:

![](attachments/Pasted%20image%2020251127112708.png)


I tried directory busting with `ffuf`: 
```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ -u http://192.168.126.135/FUZZ
```

and I found a file called `secret` and if I open it up I see this message:


![](attachments/Pasted%20image%2020251127121543.png)

So I tried the DNS instead.
