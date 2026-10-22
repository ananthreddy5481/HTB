# Touch

```
Nmap scan report for touch.htb (10.129.143.218)

PORT     STATE SERVICE
135/tcp  open  msrpc
3389/tcp open  ms-wbt-server
5985/tcp open  wsman
8443/tcp open  https-alt
```

## Port 8443(http website)

<img width="981" height="685" alt="Screenshot 2026-10-22 at 20 34 36" src="https://github.com/user-attachments/assets/ae4644bc-0c1c-4ef0-923b-b3d5fc8ecff8" />

endpoints enumeration :

```
[20:15:15] Scanning: api/
[20:15:43] 403 -   312B - /api/\..\..\..\..\..\..\..\..\..\etc\passwd       
[20:17:12] 400 -   324B - /api/Office/graph.php#xxe                         
[20:17:36] 200 -   116B - /api/status?full=true                             
[20:17:36] 200 -   116B - /api/status                                       
[20:17:36] 200 -   116B - /api/status/                                      
```

<img width="363" height="169" alt="Screenshot 2026-10-22 at 20 47 45" src="https://github.com/user-attachments/assets/8e4b3e61-fc97-4379-a267-c931d8eb5c5c" />

serial number : **NX-DH-2024-B7042** -- password for the website that is running in the 8443

<img width="1470" height="834" alt="Screenshot 2026-10-22 at 21 42 17" src="https://github.com/user-attachments/assets/0be906af-09e3-43d2-a7d9-de73e9e6685b" />

```
creds ::
Username :	KioskUser
Password : K!0sk2026#
```

## Port 3389(RDP) 
<img width="1470" height="956" alt="Screenshot 2026-10-22 at 21 42 00" src="https://github.com/user-attachments/assets/4b7e0738-7e82-456f-b039-e348de65dba4" />


here the portal is scanning for passports and visas. and the portal that control the physical machines is ```http://touch.htb:8443/printer```.

I have turned off the scanner and printer and tried to scan the staff badge and it gave a troubleshoot message giving a link which helps us to open edge browser.

<img width="1409" height="884" alt="Screenshot 2026-10-22 at 21 52 01" src="https://github.com/user-attachments/assets/94ada53b-94a4-4d95-a6bb-bd3cffd2ba37" />


through edge browser we can open command prompt.

```
C:/Windows/System32/cmd.exe
```

it will copy the the cmd.exe from /Windows/System32/ to Downloads directory and we can execute it directly.

gives command prompt and user Flag.

## User Flag
```
cca5dad1c0b1cd59c6751655cbcd6bdd
```
