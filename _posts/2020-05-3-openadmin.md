---
layout: post
title:  "Hackthebox - OpenAdmin"
date:   2020-05-3 14:07:19
categories: [Hackthebox]
excerpt: "OpenAdmin is rated as a Easy Linux box. It was released on 04 Jan 2020 and has been created by @dmw0ng

This box required us to perform the following tasks:


- Enumerate a web server to find vulnerable web application
- Exploit Web app to get initial foothold
- Credential reuse attack
- Download users SSH private key and crack
- Exploit misconfigured nano permission"

comments: true
---


## Introduction
![source-01](/img/openadmin1.png){: .align-left}

OpenAdmin is rated as a Easy Linux box. It was released on 04 Jan 2020 and has been created by @dmw0ng

This box required us to perform the following tasks:



- Enumerate a web server to find vulnerable web application
- Exploit Web app to get initial foothold
- Credential reuse attack
- Download users SSH private key and crack
- Exploit misconfigured nano permission 


## Initial reconnaissance
Let’s do first a full nmap port scan using the following command:


![source-01](/img/Screenshot_2020-05-02_09-01-51.png){: .align-left}

Two ports are open:
- SSH on 22
- Apache 2.4.29 on 80

Browsing to openadmin.htb

![source-01](/img/Screenshot_2020-05-02_09-20-39.png){: .align-left}

It just shows default apache landing page, lets enumerate this port further with gobuster to find addtional directories and files.

![source-01](/img/Screenshot_2020-05-02_09-33-18.png){: .align-left} 

Gobuster found music directory,which display a new webpage, 
![source-01](/img/Screenshot_2020-05-02_09-38-13.png){: .align-left}  

Click on login button in /music redirect to /ona
![source-01](/img/Screenshot_2020-05-02_09-38-13.png){: .align-left}  

Which is running OpenNetAdmin - v18.1.1

A quick searchsploit shows this version of OpenNetAdmin is vulnerable and there are public exploits available.

![source-01](/img/Screenshot_2020-05-02_10-04-24.png){: .align-left}  

I quickly grabbed the exploit available from exploit database.
Had weird issues running the code, due to spacing i belive (DOS-style CRLF line endings), so i converted the script using dostounix tool.
The exploit is a very simple command injection through a ping functionnality within the application.

![source-01](/img/Screenshot_2020-05-02_10-29-57.png){: .align-left}  

Running the exploit give us a shell as www-data 

![source-01](/img/Screenshot_2020-05-02_10-34-54.png){: .align-left}  

After some browsing around, I got the database settings file which contained a password.

![source-01](/img/Screenshot_2020-05-02_10-42-54.png){: .align-left}  

## Credential reuse  
	
Now lets grab the users from the system, we might be able to use the found credentials with other users
```
$ cat /etc/passwd | awk -F : '{print $1}'
root
jimmy
mysql
joanna
```
After trying the password on jimmy’s account, I was able to login to the box as Jimmy, this is a classic case of password reuse!

![source-01](/img/Screenshot_2020-05-02_14-13-04.png){: .align-left}  

And now we have a shell a jimmy user. I was excepting to get user.txt flag from here, but no we need to enumerate further more to get anywhere.
```
jimmy@openadmin:/var/www$ ls -la
total 16
drwxr-xr-x  4 root     root     4096 Nov 22 18:15 .
drwxr-xr-x 14 root     root     4096 Nov 21 14:08 ..
drwxr-xr-x  6 www-data www-data 4096 Nov 22 15:59 html
drwxrwx---  2 jimmy    internal 4096 Nov 23 17:43 internal
lrwxrwxrwx  1 www-data www-data   12 Nov 21 16:07 ona -> /opt/ona/www
```
After further enumeration, found an interesting directory under /var/wwww which belongs to jimmy user.And a php file which reads joana's RSA private keys.

![source-01](/img/Screenshot_2020-05-02_14-25-02.png){: .align-left}  

We can see that if we executed main.php it will read joanna private key.Since port 80 is serving other files, their must be virtualhost configured for this.

![source-01](/img/Screenshot_2020-05-03_08-48-47.png){: .align-left}  

From the configuration we know port is not accessible from outside , we need to run it within the local server, to grab joanna's RSA Private keys. 


Lets check other local listening ports

![source-01](/img/Screenshot_2020-05-02_14-35-51.png){: .align-left}  

We can see a service running on port 52846, so i used curl to download the content of main.php

![source-01](/img/Screenshot_2020-05-02_14-47-04.png){: .align-left}  
And here we got an encrypted RSA private key, to make this usefull we need to crack the key.

Cracking RSA key with John

![source-01](/img/Screenshot_2020-05-03_06-39-07.png){: .align-left} 

That was fast, the password for the key is bloodninjas. Now Let’s ssh into user joanna,before that remember to change permissions of the rsa key.

![source-01](/img/Screenshot_2020-05-03_06-50-07.png){: .align-left} 

We have got the user.txt

## Privillege Escalation 
Standard enumeration was enough to find the way to root. 
sudo -l command shows that user Joanna can run /bin/nano /opt/priv as the root user without entering a password.
![source-01](/img/Screenshot_2020-05-03_06-55-35.png){: .align-left} 

Reading root.txt is pretty straight forward, this technique is already well document in GTFOBins

sudo /bin/nano /opt/priv. Then we type <CTRL>+R /root/root.txt in order to read root.txt file.

OpenAdmin is rooted!








