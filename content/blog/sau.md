---
title: HTB - Sau  
pin: true
author: Scott
math: true
mermaid: true
icon: https://wolfcareers.com/wp-content/uploads/2023/01/1_cQGpZGkSuehv-YEXUweMQ-e1672674616339.webp
---


![alt text](/sau/Sau.png)


## Recon:
---

An initial `Nmap` scan reveal 3 ports open. `SSH` on port `22`, `http` on port `80` which looks like it is filtered, and it looks like `http` on port `55555` which is an unusual port for `http`.

```python
nmap -sS -sV -sC -vv -oA nmap/sau 10.10.11.224
```

![sau](/sau/1.png)



## HTTP
---
Visiting the website on port `55555`. We are presented with a page to `create new basket to inspect HTTP requests`.
![sau](/sau/2.png)
At the bottom of the page, it also leaks the version that is being used to create the website which is `1.2.1`, which could be use to further enumerate for vulnerabilities.

When we create a new basket, we are given a token with it.
![sau](/sau/3.png)

Looking at the page, we can see that there are a lot of functionalities.
![sau](/sau/4.png)

Looking at each button, we find this one to be interesting.

![sau](/sau/5.png)

Checking it, we can see that it is use to forward URL. 
![sau](/sau/6.png)
## Foothold
---
Lets examine the functionality, and see if we can access the filtered port.

![sau](/sau/7.png)

We add the localhost to the input and tick the `Proxy Response` in order to get response to the forward URL back to the client.

Now if we visit the link `http://10.10.11.224:55555/tx5fuoi`, we can see that the URL has been forwarded and we are not able to access the page that was filtered
![sau](/sau/8.png)

We can see that it leaks the version that is being used to create the website which is `0.53`.

Searching for exploit with this information, we get an exploit.
![sau](/sau/9.png)
 Let's look at the first one.
 
 We can see that using curl they are able to get RCE (Remote Code Execution). We can also see that the option `-X` is not given. `curl` uses `GET` method in default.
![sau](/sau/10.png)

So using that information lets try to exploit and get a reverse shell.

We write the reverse in a file name `index.html`. We named it `index.html` as this is the default page that is searched if no name is provided. So we don't have to keep providing a file name to get reverse shell.
![sau](/sau/11.png)

Now we open a python server as well as a listener for reverse shell.
![sau](/sau/12.png)

We configure the settings as shown in the POC (Proof of Concept). 
![sau](/sau/13.png)

Now we reload the page.

When we check the listener and python server, we can see that the page made a request to our server and we got a reverse shell back.
![sau](/sau/14.png)


We are able to read `user.txt` file.
![sau](/sau/15.png)


## Privilege Escalation
---
Lets see if the user has any `sudo` permissions.
![sau](/sau/16.png)

The user has permissions to run `sudo` without the use of password. We can see that the user can run `/usr/bin/systemctl status trail.service` with `sudo` without any password.

Let search if we can get root access using it.
![sau](/sau/17.png)


We get a page that explain how to do it so lets follow it.

![sau](/sau/18.png)


We follow the steps and we are able to get root access.

![sau](/sau/19.png)

We are able to read root.txt.

## References
---
RCE - 
[https://huntr.dev/bounties/be3c5204-fbd9-448d-b97c-96a8d2941e87/](https://huntr.dev/bounties/be3c5204-fbd9-448d-b97c-96a8d2941e87/)


Privilege Escalation - 
[https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-systemctl-privilege-escalation/](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-systemctl-privilege-escalation/)
