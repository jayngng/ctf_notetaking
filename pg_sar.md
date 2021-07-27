# Export
```md
export IP="192.168.237.35"
export URL="http://192.168.237.35:80/FUZZ"
```

<hr>

# Loot
#### Vulnerability
+ Sar2HTML 3.2.1 → Remote Code Execution
	+	PoC: https://www.exploit-db.com/exploits/47204

#### Local Enumeration
+ Local user: `love`
+ Cronjob: `*/5  *    * * *   root    cd /var/www/html/ && sudo ./finally.sh`

#### Privilege Escalation
+ Crontab

```bash
*/5  *    * * *   root    cd /var/www/html/ && sudo ./finally.sh
```


<hr>

# Nmap

```bash
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 33:40:be:13:cf:51:7d:d6:a5:9c:64:c8:13:e5:f2:9f (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDHy/WJJHLFdbwbJpTyRYhEyj2jZV024UPWIdXfNHxq45uh08jkihv3znZ98caLP/pz352c0ZYD31We0bTSbHyjQce2bSAJHubDYp13hU/P4tbV5GIJ72W2rWkLTslH/SJoHUSqlManB7ZzgVyU2KQ4fnNx/V1XGJYsshquRqTrXKeeal+yQvTC4gnsr8ENIGMq0yJnYxMAasx6kmSc+S+065Mie65xkyisFXo2MQyxzsFdCu2w1bYmb3pegYDm6Y0c/EJP0sxDizXVwkUOS0XSVdGuk3RUYjt5GQ2fL24ZsML6CwN+HD2ZTnD0FK90PQTLuvlp6BoI/ZWvIenNvu63
|   256 8a:4e:ab:0b:de:e3:69:40:50:98:98:58:32:8f:71:9e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFgxutbLnN4K2tj6ZHzrlzTKS+RRuly+RkA0J63JsQFiwyvz4PqA64w/h0Se3gymZV6zJ9XBpS41b6IoEymeiSA=
|   256 e6:2f:55:1c:db:d0:bb:46:92:80:dd:5f:8e:a3:0a:41 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIM+5254x35Vwa2S7X73YLY87Q58qQOD9oQeSKMpmmT0o
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

<hr>

# Web Application
#### Port 80
+ `/robots.txt` reveals `sar2HTML` 
+ `/sar2HTML` is running `sar2html Ver 3.2.1`
