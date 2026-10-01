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

after connecting to the wifi provided by the
