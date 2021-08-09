# Export
```md
export IP="192.168.69.76"
export URL="http://192.168.69.76:80/FUZZ"
```

<hr>

# Remote Code Execution
+ Koken
```bash
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                              |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Koken CMS 0.22.24 - Arbitrary File Upload (Authenticated)                                                                                                   | php/webapps/48706.txt
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
```

<hr>

# Credentials Panel
+ (Web 8080): `daisa@photographer.com:babygirl`


<hr>

# Privilege Escalation
+ SUID sticky bit
+ GTFOBins
```md
https://gtfobins.github.io/gtfobins/php/
```

<hr>

# Nmap
```bash
PORT     STATE SERVICE     REASON         VERSION
22/tcp   open  ssh         syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:    
|   2048 41:4d:aa:18:86:94:8e:88:a7:4c:6b:42:60:76:f1:4f (RSA)          
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCq9GoYsvJTOUcsgHSES9+20Ix4Q8wjm5slMheJ2ME+COokAqxBzXSr458KBmHv3bsTLWAH9FxoXJ6zrzDPmPApcqVifB4aI9l/VYxoeJCj54kKIQlCKkWTZjsAeLBI2Lk2+yJLLFWPTAZ2htwRAwCl
9z8YV3xgtqhTa+5BqIm/GInW4PYV0zi9zOMn2g4jNSWvy91FBUboGLwVgNYslGBydNW8Fhz8X/LXHZ1x6ulA76W026VEGOiQfoiIi84IFi9CbP8GIKfQ7BHuDlMqgiN9+w7K0z0oFdtiFhAS/48w89MYn6UOzw7Aaa9eLQi0+zxpW5SpCpw0mC2euzPxow
2Z
|   256 4d:a3:d0:7a:8f:64:ef:82:45:2d:01:13:18:b7:e0:13 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMz4UG2gfu7L/Lxcqek1pZf46d8SocbES1A2a/XUYQgTmIqJuCEpLf3ERgVXS+7Lwdi6+F3xkI/lYFCA5MkRUQA=
|   256 1a:01:7a:4f:cf:95:85:bf:31:a1:4f:15:87:ab:94:e2 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDL5ZwzA5dpqtWx4ZzjVQ6NMzVUia8/We8txfiAn+mv4
80/tcp   open  http        syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)   
|_http-title: Photographer by v1n1v131r4
139/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
8000/tcp open  http        syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: Koken 0.22.24            
| http-methods:      
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: daisa ahomi
```

<hr>

# Web Application
#### Port 80
+ Discovery

```bash
.htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10]
.hta                    [Status: 403, Size: 278, Words: 20, Lines: 10]
assets                  [Status: 301, Size: 315, Words: 20, Lines: 10]
images                  [Status: 301, Size: 315, Words: 20, Lines: 10]
index.html              [Status: 200, Size: 5711, Words: 296, Lines: 190]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10]
```

+ User?: `v1n1v131r4`

#### Port 8000
+ Discovery
```bash
.htaccess               [Status: 403, Size: 280, Words: 20, Lines: 10]
.htpasswd               [Status: 403, Size: 280, Words: 20, Lines: 10]
.hta                    [Status: 403, Size: 280, Words: 20, Lines: 10]
admin                   [Status: 301, Size: 321, Words: 20, Lines: 10]
app                     [Status: 301, Size: 319, Words: 20, Lines: 10]
index.php               [Status: 200, Size: 4603, Words: 206, Lines: 95]
server-status           [Status: 403, Size: 280, Words: 20, Lines: 10]
storage                 [Status: 301, Size: 323, Words: 20, Lines: 10]
```

+ `/admin` is running "Koken" CMS?

<hr>

# SMB
+ File shares
```md
Enter WORKGROUP\root's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        sambashare      Disk      Samba on Ubuntu
        IPC$            IPC       IPC Service (photographer server (Samba, Ubuntu))
```

+ Anonymous access.
+ `sambashare` file shares:

```md
smb: \> ls
  .                                   D        0  Thu Aug 20 11:51:08 2020
  ..                                  D        0  Thu Aug 20 12:08:59 2020
  mailsent.txt                        N      503  Mon Jul 20 21:29:40 2020
  wordpress.bkp.zip                   N 13930308  Mon Jul 20 21:22:23 2020

                3300080 blocks of size 1024. 2958792 blocks available
```

+ `mailsent.txt` content → Valid gmail: `daisa@photographer.com`

```md
...
To: Daisa Ahomi <daisa@photographer.com>
...
```
