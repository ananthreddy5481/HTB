# Support #

## Enumeration ##

<img width="669" height="395" alt="Screenshot 2026-05-11 at 19 22 28" src="https://github.com/user-attachments/assets/f90a305b-4474-442e-ae0e-023999365f3c" /><br>

-Pn -> force scan of the ports for services.

```port 139 & 445``` - smb

### smb

server message block - network protocol which is used to share files on local network.

Share - the folder in the service exposed by the system present in the local network. each system can have multiple shares.

<img width="632" height="266" alt="Screenshot 2026-05-11 at 20 06 23" src="https://github.com/user-attachments/assets/4c3916db-5cea-4c06-9e53-8405454cb753" /><br>

non default share -- ```support-tools```


connecting to a share 

```
smbclient -N //support.htb/support-tools
```

<img width="1187" height="385" alt="Screenshot 2026-05-11 at 20 13 45" src="https://github.com/user-attachments/assets/e6a0ca84-542b-47a3-a107-b48481cb3985" /><br>


After unzipping the file ```UserInfo.exe``` we got some ```.dll``` files and ```UserInfo.exe``` an executable file and ```config``` file .

<img width="1051" height="386" alt="Screenshot 2026-05-11 at 20 28 47" src="https://github.com/user-attachments/assets/6a8a0046-7832-4d4f-8004-342d587bfdb8" /><br>


**unable to execute the executible file due to binary content.**


### AD credentials

After Decompiling the executable file  :

In ```/UserInfo.Services/LdapQuery.cs``` we get username .

In ```/UserInfo.Services/protected.cs``` we get the password .

```
Username :

hashed Password : 0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E

key : armando

Original Password : <img width="685" height="644" alt="Screenshot 2026-06-13 at 22 41 24" src="https://github.com/user-attachments/assets/31457166-459d-4ca8-9337-307bf48d80c3" />
```

from the line (LdapQuery.cs) :
```
entry = new DirectoryEntry("LDAP://support.htb", "support\\ldap", password);
```

LDAP --> protocol to communicate with AD.

support.htb --> domain name --> name used to identify the AD in the local network .

support --> name used by NETBios service to identify the domain of the user.

ldap --> username of the AD (Active directory).

### NETBios 

the protocol used to find the machines in the local network just like DNS protocol which finds machines in the wide network.

### LDAP 

**Lightweight Directory Access Protocol**

used to  communicate with the directory sharing services like AD using the ports 389 and 636.

```winRM``` -- Windows Remote Management -- operated on port 5985

It is a Microsoft service that allows you to remotely execute commands and manage Windows machines over the network.

<img width="1305" height="348" alt="Screenshot 2026-06-13 at 21 55 21" src="https://github.com/user-attachments/assets/0506229d-5816-472c-9c13-38b10464608c" />

```ldap``` user does not have the authorization for the winRM shell .

```ldapsearch``` 

command tool used to communicate with the AD using LDAP .

```
ldapsearch -x -H ldap://support.htb \ -D "SUPPORT\\ldap" \ -w 'password' \ -b "DC=support,DC=htb" \ "(objectClass=user)" \
sAMAccountName description
```


