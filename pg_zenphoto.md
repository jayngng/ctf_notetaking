# Export
```md
export IP="192.168.237.41"
export URL="http://192.168.237.41/FUZZ"
export LIP="192.168.49.237"
```

<hr>

# Loot
+ `zenphoto version 1.4.1.4` CMS running on HTTP port 80.
+ Also a **VULNERABLE CMS** version →**Remote Code Execution**.
	+ Exploit code: https://www.exploit-db.com/exploits/18083

#### Local Enumeration → Privilege Escalation

+ (MYSQL) Credentials: `root:hola`
+ Is there any local user on the system? NO
+ Is the kernel out-of-date? YES `Linux offsecsrv 2.6.32-21-generic #32-Ubuntu SMP Fri Apr 16 08:10:02 UTC 2010 i686 GNU/Linux` → **VULNERALBE** to DirtyCow → Pwn!.
	+ Exploit code: https://www.exploit-db.com/exploits/40839

<hr>

# Nmap

```bash
PORT     STATE SERVICE REASON         VERSION                                                                                                                                                                      
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 5.3p1 Debian 3ubuntu7 (Ubuntu Linux; protocol 2.0)                                                                                                                   
| ssh-hostkey:                                                                                                                                                                                                     
|   1024 83:92:ab:f2:b7:6e:27:08:7b:a9:b8:72:32:8c:cc:29 (DSA)                                                                                                                                                     
| ssh-dss AAAAB3NzaC1kc3MAAACBAIM3Qmxj/JapoH/Vg/pl8IAj0PTqw5Fj5rnhI+9Q0XT5tej5pHpUZoWTmbQKIwA7QBoTWtk4Hnonhkv5We43VXz0abBEvy3allgjf13cvxc96KX0bE7Bb8PhVCQJJBDTIz44koJhvFuSO/sauL9j+lzaUltVMR6/bZbigTINrV4nAAAAFQCvl
Vi2Us40FGWv8TILJYOR/LJvcwAAAIAHpp8VGuPUA5BowTa55myGr/lGs0xTFXbxFm0We4/D5v3L9kUVgv6MIVL4jweRmXFYvei7YZDGikoe6OjF9PFtSkKriEaGqav6hOER3tmtWChQfMlaNwiZfNJzKHBc4EqeCX4jpLLUxCZAEjwoE0koQRoFcbr+gywBNOQgtrfv+QAAAIA8v2C1
COdjtNl4Bp3+XVLOkbYPIpedQXCgTLgRloa5wQZCaZimgE3+txqTQSb7Vp0B+LfjKdqcMFia8g9i+0YC+b69NimiFaZXU8euBoh/GXNo8K2vFHF3yznq6KNPG4+EW3WfaLGqJWkBJM2bb1nJ0YaJZhpOInv2Gsanh4CHOA==                                           
|   2048 65:77:fa:50:fd:4d:9e:f1:67:e5:cc:0c:c6:96:f2:3e (RSA)                                                                                                                                                     
|_ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA7aKskCBM7hdQEibRza0Y1BAiJ0prjECzVow5/txHOHb+Ynokd1ByaBw5roKsOExD3h7d7VGjNVKNqSwB+SBHSRivJaEgCtiV3F/5Q1qdBpehE4zyv7whG9GKeALeNk05icqXCk9kveUsreZyqEqN+c9p3Ed29jTD+6Alc7mml/Zev0EQs7hFfX/kYiV6V4KnQuQ7HXe3kzbMA9WB3yxtp0saBB5zlu4eWGsvyvCibP41ce81LtwkJDSXTr0LwBNYgZOD07GWW//BkOuJvHtKbWPqBievO0yubQxGbz0r7vID3a5DQMj4ZTGrAQPCunaJkGlvZs2zftrUh/BMxQSFLw==
23/tcp   open  ipp     syn-ack ttl 63 CUPS 1.4
| http-methods: 
|   Supported Methods: GET HEAD OPTIONS POST PUT
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/1.4
|_http-title: 403 Forbidden
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.2.14 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.14 (Ubuntu)
|_http-title: Site doesn't have a title (text/html). 
3306/tcp open  mysql   syn-ack ttl 63 MySQL (unauthorized)
```


<hr>

# Web Application
### Port 80

+ Discovery

```md
.htpasswd               [Status: 403, Size: 291, Words: 21, Lines: 11]
.htaccess               [Status: 403, Size: 291, Words: 21, Lines: 11]
.hta                    [Status: 403, Size: 286, Words: 21, Lines: 11]
cgi-bin/                [Status: 403, Size: 290, Words: 21, Lines: 11]
index                   [Status: 200, Size: 75, Words: 2, Lines: 5]
index.html              [Status: 200, Size: 75, Words: 2, Lines: 5]
test                    [Status: 301, Size: 315, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 295, Words: 21, Lines: 11]
```

+ `/test/index.php` leaks CMS application and version used: `zenphoto version 1.4.1.4`

```md
view-source:http://192.168.237.41/test/index.php

<!-- zenphoto version 1.4.1.4 [8157] (Official Build) THEME: default (index.php) GRAPHICS LIB: PHP GD library 2.0 { memory: 128M } PLUGINS: class-video colorbox deprecated-functions hitcounter security-logger tiny_mce zenphoto_news zenphoto_sendmail zenphoto_seo  --> 
<!-- Zenphoto script processing end:0.0811 seconds -->
```