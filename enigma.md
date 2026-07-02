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


