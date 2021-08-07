# Export
```md
export IP="10.10.190.154"
export URL="http://10.10.190.154:80/FUZZ"
```

<hr>

# Vulnerability
+ SMB file share allows anonymous access.
+ **Base64-encoded** passwords.
+ RCE 1: EternalBlue with `bob`'s credentials' → Administrator. (This method returns a semi-interactive shell, which is extremely unstable, need further persistence.)
+ RCE 2: Upload shell on SMB share → Access the shell with one of the web service.
+ Misconfugred privilege token `SeImpersonatePrivilege` → Privilege Escalation using `PrintSpoofer`.

<hr>

# Credentials Panel
+ `Bob - !P@$$W0rD!123`
+ `Bill - Juw4nnaM4n420696969!$$$`

<hr>

# Persistence
+ Enable `ADMIN$` share writable by everyone.

```bash
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\system /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```

→ `psexec.py` to obtain interactive shell.

<hr>

# Nmap
```bash
PORT      STATE SERVICE       REASON          VERSION
80/tcp    open  http          syn-ack ttl 125 Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp   open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 125 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  syn-ack ttl 125 Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open  ms-wbt-server syn-ack ttl 125 Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2021-08-07T10:16:18+00:00
| ssl-cert: Subject: commonName=Relevant
| Issuer: commonName=Relevant
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-08-06T09:13:32
| Not valid after:  2022-02-05T09:13:32
| MD5:   cd39 25ee bdc8 3869 479a 119e 17e0 44b9
| SHA-1: 8302 b9c7 b55f 9581 7900 d696 924d 162a 527c 5d96
| -----BEGIN CERTIFICATE-----
| MIIC1DCCAbygAwIBAgIQGO/8WZ/trLhMbv41pDcJNjANBgkqhkiG9w0BAQsFADAT
| MREwDwYDVQQDEwhSZWxldmFudDAeFw0yMTA4MDYwOTEzMzJaFw0yMjAyMDUwOTEz
| MzJaMBMxETAPBgNVBAMTCFJlbGV2YW50MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
| MIIBCgKCAQEApplWW/v/qOWSDENL4ddZoIvJGUe1wrndDbjq6ObZ1dzz67k8wUyz
| 1DWObAmwIFGyntzIEyL88PqtU+eiFp6F75jX+a/hb4ebprLJ3G2nEFk/6x5WY7Vb
| 8vcYfwKqWC0jId7JTbJWm5En/0KzGex+ur/k5FCy0HYpwRnWlx503D8aJ5hPmyyq
| FVJ0emQ67BqiSiJ/mLm377JYV4tdfY0jEcLAlgTakKyjz9vkj4hxK1ZcVafbItdG
| GHDSEztv6pH1C0BNgHW2dQiwYF5O3WvsjOuQfX3NkjpM12dt84l6V1nNaMiQabsO
| 4yfADA/lkC3uAolb6K2xLh0BHwgrte8kuwIDAQABoyQwIjATBgNVHSUEDDAKBggr
| BgEFBQcDATALBgNVHQ8EBAMCBDAwDQYJKoZIhvcNAQELBQADggEBAI14oWPpOT0L
| cw+gqHB4J45QqSahy8u4nozFv8vo58tIrqnG474s7gdLPqUDpe0jW3WvJGSknXpE
| hcdtUs31KRSLnz/Y4GXZgEQWLkFOdH3vxxw629a74APrJsZhO4IhqLUZeouI1P4K
| zc3e7unlh2ufX40EgOrz1h9hqj/MC6kyunBihEl+0xZqmqNClLB5shqIdxVvDY5F
| 6V/Y7qUGictPmKztOQvPLAllPTceSkCwdQZ2vDfFuJQt8X2Pu7bCI92vAEw3GtjI
| 2PZ3M6lUyHiIjeJwloZ0fcyOwlD33laCGtU0gaaXIIX2OeF6GmwU2EQcf/Lr8XuP
| p6B8Nt9BwSI=
|_-----END CERTIFICATE-----
|_ssl-date: 2021-08-07T10:16:58+00:00; 0s from scanner time.
49663/tcp open  http          syn-ack ttl 125 Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
49667/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
```


<hr>

# Web Application
#### Port 80
+ Discovery
+ Nothing is really interesting

#### Port 49663
+ Able to directly access SMB file share

```md
http://10.10.190.154:49663/nt4wrksv/passwords.txt

[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```

<hr>

# SMB
+ Allows anonymous login
+ Files shares

```bash
Enter WORKGROUP\root's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk
```

+ The `nt4wrksv` share contains the file `passwords.txt`.

```bash
smb: \> ls
  .                                   D        0  Sat Jul 25 17:46:04 2020
  ..                                  D        0  Sat Jul 25 17:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020

                7735807 blocks of size 4096. 5142794 blocks available
```

+ Directory `nt4wrksv` is writable.

<hr>

# RDP
+ Try to login with found creds but failed.

