# Export
```md
export IP="192.168.237.67"
export URL="http://192.168.237.67:3000/FUZZ"
```

<hr>

# Loot
#### Vulnerability
+ Gitea v1.7.5 is vulnerable to Remote Code Execution.
	+	https://www.exploit-db.com/exploits/49383

#### Local Enumeration
+ Crontab
+ `/usr/bin/local` → writable → Privilege Escalation via PATH hijacking

<hr>

# Nmap
```md
PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 63 ProFTPD 1.3.5b
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 
2222/tcp open  ssh     syn-ack ttl 63 Dropbear sshd 2016.74 (protocol 2.0)
3000/tcp open  ppp?    syn-ack ttl 63
| fingerprint-strings: 
|   GenericLines, Help: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gitea=a8bd8ac7bb76a6cc; Path=/; HttpOnly
|     Set-Cookie: _csrf=rWwE_aKxVG0zA3Hb7Qv6y_Fn7MU6MTYyNzM2OTg2NTQwMTEyMTkxNQ%3D%3D; Path=/; Expires=Wed, 28 Jul 2021 07:11:05 GMT; HttpOnly
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 27 Jul 2021 07:11:05 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="/manifest.json" crossorigin="use-credentials">
|     <script>
|     ('serviceWorker' in navigator) {
|     window.addEventListener('load', function() {
|     navigator.serviceWorker.register('/serviceworker.js').then(function(registration) {
|   HTTPOptions: 
|     HTTP/1.0 404 Not Found
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gitea=08ff968f313f3712; Path=/; HttpOnly
|     Set-Cookie: _csrf=2unrSZxevOhoZ9pcrgUXPaGK8Kc6MTYyNzM2OTg3MjIzMzQ2MDczOQ%3D%3D; Path=/; Expires=Wed, 28 Jul 2021 07:11:12 GMT; HttpOnly
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 27 Jul 2021 07:11:12 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>Page Not Found - Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="/manifest.json" crossorigin="use-credentials">
|     <script>
|     ('serviceWorker' in navigator) {
|     window.addEventListener('load', function() {
|_    
```

<hr>

# Web Application
#### Port 3000
+ Gitea: self-hosted Git service. It is similar to GitHub, Bitbucket, and GitLab.
+ Valid user: `admin`
+ Version: `Gitea v1.7.5` → Remote Code Execution
+ Discovery
```md
admin                   [Status: 302, Size: 34, Words: 2, Lines: 3]
debug                   [Status: 200, Size: 160, Words: 18, Lines: 5]
explore                 [Status: 302, Size: 37, Words: 2, Lines: 3]
issues                  [Status: 302, Size: 34, Words: 2, Lines: 3]
notifications           [Status: 302, Size: 34, Words: 2, Lines: 3]
```

<hr>

# FTP
+ NO anonymous login
+ ProFTPD 1.3.5b → Vulnerable version but not exploitable

<hr>

# SSH
#### Port 2222
+ Dropbear sshd 2016.74


#### Port 22
+ OpenSSH 7.4p1
