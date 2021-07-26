# Export
```bash
export IP="192.168.237.66"
export URL="http://192.168.237.66:80/FUZZ"
```

<hr>

# Loot
#### Vulnerability
+ H2 Database 1.4.199 
	+ PoC: https://www.exploit-db.com/exploits/49384



#### Credentials
+ H2 Console (Web): `sa:(no password)` → Administrator log in

#### Local Enumeration
+ WinPEASx64 results:

```bash
 [!] CVE-2020-1013 : VULNERABLE
  [>] https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/

 [!] CVE-2020-0796 : VULNERABLE
  [>] https://github.com/danigargu/CVE-2020-0796 (smbghost)

 [*] Finished. Found 2 potential vulnerabilities.

=-=-=-=-=-=-=-=-=

C:\Users\tony\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
=-=-=-=-=-=-=-=-=

SeImpersonatePrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
=-=-=-=-=-=-=-=-=

(DLL Hijacking) C:\JavaTemp\: Authenticated Users [WriteData/CreateFiles]           =-=-=-=-=-=-=-=-=

File: C:\Program Files (x86)\Common Files\Java\Java Update\jusched.exe (Unquoted and Space detected)

=-=-=-=-=-=-=-=-=
  Version: NetNTLMv2              
  Hash:    tony::JACKO:1122334455667788:36affa6363992f068ff1721eeac3a4e3:01010000000000007b6959501e82d701f7273b82d62c5a36000000000800300030000000000000000000000000300000c854ea0a8f4dcdd8bb420
1a7ed1c553cc740fa87dbd6763c7063e1ec205803dd0a00100000000000000000000000000000000000090000000000000000000000
```

#### Privilege Escalation
+ Abuse token `SeImpersonatePrivilege` w/ `PrintSpooferx64.exe`
	+ https://github.com/itm4n/PrintSpoofer 

<hr>

# Nmap
```bash
PORT     STATE SERVICE       REASON          VERSION
80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods:                                                                                
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0      
|_http-title: H2 Database Engine (redirect)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds? syn-ack ttl 127
7680/tcp open  tcpwrapped    syn-ack ttl 127
8082/tcp open  http          syn-ack ttl 127 H2 database http console
|_http-favicon: Unknown favicon MD5: D2FBC2E4FB758DC8672CDEFB4D924540
| http-methods:             
|_  Supported Methods: GET POST
|_http-title: H2 Console  
```

<hr>

# Web Application
#### Port 80
+ Discovery

```bash
Images                  [Status: 301, Size: 155, Words: 9, Lines: 2]
HTML                    [Status: 301, Size: 153, Words: 9, Lines: 2]
Help                    [Status: 301, Size: 153, Words: 9, Lines: 2]
help                    [Status: 301, Size: 153, Words: 9, Lines: 2]
html                    [Status: 301, Size: 153, Words: 9, Lines: 2]
images                  [Status: 301, Size: 155, Words: 9, Lines: 2]
javadoc                 [Status: 301, Size: 156, Words: 9, Lines: 2]
text                    [Status: 301, Size: 153, Words: 9, Lines: 2]
```


#### Port 8082

+ H2 Console
+ H2 v1.4.199 
+ Discovery

```bash
favicon.ico             [Status: 200, Size: 4286, Words: 6, Lines: 50]
index.html              [Status: 200, Size: 937, Words: 89, Lines: 25]
```