# Export
```bash
IP="192.168.130.40"
URL="http://192.168.130.40::80/FUZZ"
```

<hr>

# Nmap
```bash
$ sudo nmap -p- -sV -T4 192.168.130.40
PORT STATE SERVICE VERSION
53/tcp open domain Microsoft DNS 6.0.6001 (17714650) (Windows Server 2008 SP1)
135/tcp open msrpc Microsoft Windows RPC
139/tcp open netbios-ssn Microsoft Windows netbios-ssn
445/tcp open microsoft-ds Microsoft Windows Server 2008 R2 microsoft-ds (workgroup: WORKGROUP)
763/tcp filtered cycleserv
1147/tcp filtered capioverlan
3389/tcp open ssl/ms-wbt-server?
5357/tcp open http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
...
```