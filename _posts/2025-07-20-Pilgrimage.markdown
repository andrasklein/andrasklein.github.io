---
layout: post
title: Pilgrimage
date: 2025-07-20 10:28:18 +0800
categories: [HackTheBox, Easy]
tags: [Git, PNG, SQLite, CVE-2022-4510, Binwalk]
author: klein
image:
  path: /images/images-pilgrimage/image_11.png
---

This notes-style walkthrough will be conducted using HackTheBox Guided Mode for this retired machine.

### Which version of nginx does the target machine run on TCP port 80?

This can be enumerated with the help of an Nmap scan. 

```bash
 nmap -sC -sV -oN pilgrimage 10.129.181.211 -vv
```
```bash

80/tcp open  http    nginx 1.18.0
|_http-title: Pilgrimage - Shrink Your Images
|_http-server-header: nginx/1.18.0
| http-git:
|   10.129.181.211:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: Pilgrimage image shrinking service initial commit. # Please ...
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


```
### What is the name of the directory related to version-control software that is exposed by the web server?

Let's look at the web portal to find this information.
The IP address redirects to pilgrimage.htb. Let's add this to the hosts file.

![b](/images/images-pilgrimage/image_01.png)

```bash
 gobuster dir -u http://pilgrimage.htb/. -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt
```
By directory brute forcing, we find the .git directory. 

![b](/images/images-pilgrimage/image_02.png)

### What version of the tool ImageMagick is contained in the web application's repository?

Let's create a Git directory and use the git-dumper tool to get started.

```bash
git-dumper http://pilgrimage.htb/.git git/
```
By looking around, we find a binary named "magic". By running this magic binary, we can determine its version.

![b](/images/images-pilgrimage/image_03.png)

### What is the 2022 CVE ID for a file read vulnerability in this version of ImageMagick?

By doing a quick Google search, we can easily find the CVE ID for the file read vulnerability.

![b](/images/images-pilgrimage/image_04.png)

### Where does the web application store its data?

By taking a look at the source code, we can quickly find that it is stored in an SQL database. I preferred to open the files in Visual Studio Code and find the file path that way.

![b](/images/images-pilgrimage/image_05.png)

### What is the Emily user's password?

By finding a POC exploit (https://github.com/Sybil-Scan/imagemagick-lfi-poc), I was able to extract the user's password.

![b](/images/images-pilgrimage/image_06.png)

### What is the full path to the unusual bash script being run by the root user?

By looking at the running processes.
```bash
ps -ef --forest | less -S
```
We find the following script that is running.

![b](/images/images-pilgrimage/image_07.png)

### Which directory is monitored by inotifywait and passes files to binwalk? (Include trailing /).

By taking a look at the bash script, we can find the directory that is monitored by inotifywait.

![b](/images/images-pilgrimage/image_08.png)


### What is the 2022 CVE ID for a remote code execution (RCE) vulnerability in this version of binwalk?

By running the command binwalk, we get exposed to the version of binwalk, which in this case is Binwalk v2.3.2.

![b](/images/images-pilgrimage/image_09.png)

By doing a quick Google search, we find the CVE-2022-4510, which is going to get us to root. And by following the steps and downloading the exploit in the right directory, we get back a reverse shell.

![b](/images/images-pilgrimage/image_10.png)

I used this exploit by the way: https://www.exploit-db.com/exploits/51249.


