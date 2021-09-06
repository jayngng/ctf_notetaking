# Export
```bash
IP="192.168.106.69"
URL="http://192.168.106.69:80/FUZZ"
```

<hr>

# RCE
+ Redis 5.0.9 → Unauthenticated RCE → root


<hr>

# Nmap
```bash
$ sudo nmap --open -sV -A -p- -vv -n -Pn $IP
PORT STATE SERVICE REASON VERSION
22/tcp open ssh syn-ack ttl 63 OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
6379/tcp open redis syn-ack ttl 63 Redis key-value store 5.0.9
8080/tcp open http-proxy syn-ack ttl 63
27017/tcp open mongod? syn-ack ttl 63
```

