---
title: HTB - AppSanity
pin: true
layout: ""
author: Scott
math: true
date: 
mermaid: true
icon: https://wolfcareers.com/wp-content/uploads/2023/01/1_cQGpZGkSuehv-YEXUweMQ-e1672674616339.webp
---

# Overview:
Appsanity is a  Windows Hard Machine. 



![alt text](/appsanity/photo.png)

Machine Author: xRogue                                                                                              
Was Released on : 28 Oct 2023 

## Recon

### Nmap:

As always start with port scanning of AppSanity using nmap:

```python
kali Hard/AppSanity » sudo nmap -sC -sV 10.10.11.238 -vv -p- --min-rate 5000  -oN nmap/initial 
Nmap scan report for 10.10.11.238 (10.10.11.238)
Host is up, received echo-reply ttl 127 (0.28s latency).
Scanned at 2024-03-08 20:56:40 +0545 for 130s
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE    REASON          VERSION
80/tcp   open  http       syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to https://meddigi.htb/
|_http-server-header: Microsoft-IIS/10.0
443/tcp  open  https?     syn-ack ttl 127
7680/tcp open  pando-pub? syn-ack ttl 127
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar  8 20:58:50 2024 -- 1 IP address (1 host up) scanned in 130.73 seconds
```

The nmap result listed out the different ports i.e 80, 443 and 7680.
 <br>
 
The TCP Port 80 didnot follow redirection. So adding `meddigi.htb` at /etc/hosts.
![alt text](/appsanity/host.png)

### HTTP 443
This page is related to health and services.
![alt text](/appsanity/443.png)

looks like there is login page . 
![alt text](/appsanity/login.png)

Register First,
