
We perform a `nslookup` to find some domain name:

![](attachments/Pasted%20image%2020251127123602.png)

and we find this domain name so we put it to `/etc/hosts` with:

```bash
echo "192.168.126.135 blackpearl.tcm" >> /etc/hosts
```

and now if we visit `http://blackpearl.tcm` we find this page:

![](attachments/Pasted%20image%2020251127123721.png)

