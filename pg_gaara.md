# Export
```md
export IP="192.168.77.142"
export URL="http://192.168.77.142:80/FUZZ"
```

<hr>

# Loot

#### Credentials
+ SSH: `gaara:iloveyou2`

#### Privilege Escalation
+ SUID binary.

<hr>

# Nmap
```bash
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.38 ((Debian))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Gaara
```

<hr>

# Web Application
+ Discovery:

```md
index.html              [Status: 200, Size: 137, Words: 40, Lines: 6]
.htaccess               [Status: 403, Size: 279, Words: 20, Lines: 10]
.                       [Status: 200, Size: 137, Words: 40, Lines: 6]
.html                   [Status: 403, Size: 279, Words: 20, Lines: 10]
.htpasswd               [Status: 403, Size: 279, Words: 20, Lines: 10]
.htm                    [Status: 403, Size: 279, Words: 20, Lines: 10]
.htpasswds              [Status: 403, Size: 279, Words: 20, Lines: 10]
.htgroup                [Status: 403, Size: 279, Words: 20, Lines: 10]
.htaccess.bak           [Status: 403, Size: 279, Words: 20, Lines: 10]
.htuser                 [Status: 403, Size: 279, Words: 20, Lines: 10]
```

+ Potential username: `gaara`
