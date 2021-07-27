# Export
```md
export IP="192.168.234.18"
export URL="http://192.168.234.18:80/FUZZ"
```

<hr>

# Loot 
+ @fowsniff twitter account leaks multiple password hashes.

#### Credentials
+ (pop3): `seina:scoobydoo2`
+ (ssh): `baksteen:S1ck3nBluff+secureshell`

#### Local Enumeration
+ `/opt/cube/cube.sh` is writable to the `users` group → SSH our way into the box will automatically trigger this script as `root`.

<hr>

# Nmap
```md
PORT    STATE SERVICE REASON         VERSION
22/tcp  open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu 
80/tcp  open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Fowsniff Corp - Delivering Solutions
110/tcp open  pop3    syn-ack ttl 63 Dovecot pop3d
|_pop3-capabilities: USER CAPA RESP-CODES SASL(PLAIN) AUTH-RESP-CODE UIDL PIPELINING TOP
143/tcp open  imap    syn-ack ttl 63 Dovecot imapd
|_imap-capabilities: LOGIN-REFERRALS AUTH=PLAINA0001 ENABLE more IMAP4rev1 capabilities ID post-login have SASL-IR Pre-login listed OK IDLE LITERAL+
```

<hr>

# Web Application
+ Discovery

```md
images                  [Status: 301, Size: 317, Words: 20, Lines: 10]
assets                  [Status: 301, Size: 317, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 302, Words: 22, Lines: 12]
```
