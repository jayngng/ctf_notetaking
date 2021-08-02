# Export
```md
export IP="192.168.111.156"
export URL="http://192.168.111.156:80/FUZZ"
```

<hr>

# Hints
+ The website is vulnerable to SIP digest leak.
+ Decode audio sound.

<hr>

# Loot
+ The  `sipdigestleak.pl` tool extract password hash utilising challenge/response vulnerability in the current SIP version. 


#### Credentials
→`/login.php` port 8000: `admin:admin`
→ `/login.php` port 80: `adm_sip:passion`
→ SSH: `voiper:Password1234`

#### Privilege Escalation
+ User `voiper` can run sudo.

<hr>

# Take away
+ If the VoIP responder has configuration `<recv response="**">` → might be vulnerable to SIP digest leak because it allows us to send `407 Proxy Auth Required` instead of `ACK`, which will trigger the digest leak response.
+ We need a correct encoding algorithm to decode RTP `.raw` audio file, such as:
	+	`frequency: 8000Hz`
	+	`encoding: G.711 pcm_mulaw`
	+ We can then use `sox` to recover the original audio sound.

<hr>

# Nmap
```bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-favicon: Unknown favicon MD5: 7D4140C76BF7648531683BFA4F7F8C22
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-title: VOIP Manager
|_Requested resource was login.php
8000/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_Requested resource was login.php
```

<hr>

# Web Application
#### Port 80
+ Discovery

```md
.htpasswd               [Status: 403, Size: 280, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 280, Words: 20, Lines: 10]
files                   [Status: 301, Size: 318, Words: 20, Lines: 10]
login                   [Status: 301, Size: 318, Words: 20, Lines: 10]
```

+ Login with creds: `adm_sip:passion`
+ `http://192.168.111.156/cdr.php` has a raw audio file RTPplay1.0?
+ `/streams.php` reveals audio encoding method.

Decode:
```
sox -t raw -r 8000 -v 4 -c 1 -e mu-law 2138.raw out.wav
```

+ Listening to the `out.wav` audio file reveals a new password: `Password1234`

#### Port 8000
+ VOIP v2.0
+ Hardware Version: 3302BC
+ Login: `admin:admin`
+ `/users.php` reveals potential users
```md
william
emma
voiper
john
olivia
ava
rocky → Disable
noah → Disable
```


<hr>

# Researches
+ `#!rtpplay1.0`  header in the `2138.raw` was created using `rtpdump -F dump` ? 
	+ http://www.cs.cmu.edu/~./libra-demo/rtptools-1.17/rtptools.html#rtpplay

+ If use `rtpplay`, we can read the RTP session data recorded from `rtpdump -F dump` 
