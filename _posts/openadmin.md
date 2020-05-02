## Introduction
![source-01](/img/openadmin1.png){: .align-left}

OpenAdmin is rated as a Easy Linux box. It was released on 04 Jan 2020 and has been created by @dmw0ng

This box required us to perform the following tasks:


- Enumerate a web server to find vulnerable web application


## Initial reconnaissance
Let’s do first a full nmap port scan using the following command:
> nmap -sC -sV -oA nmap/initial 

![source-01](/img/Screenshot_2020-05-02_09-01-51.png){: .align-left}

We have SSH on 22, and an Apache 2.4.29 HTTP server running on 80

Lets see what is running on port 80

![source-01](/img/Screenshot_2020-05-02_09-20-39.png){: .align-left}

It just shows default apache landing page, lts enumerate this port further with gobuster.

![source-01](/img/Screenshot_2020-05-02_09-33-18.png){: .align-left} 

Gobuster found music directory,which display a new webpage, 
![source-01](/img/Screenshot_2020-05-02_09-38-13.png){: .align-left}  

from there when we click login it redirect to /ona
![source-01](/img/Screenshot_2020-05-02_09-40-35.png){: .align-left}  

Which is running OpenNetAdmin - v18.1.1

A quick searchsploit shows this version of OpenNetAdmin is vulnerable and there are public exploits available.

![source-01](/img/Screenshot_2020-05-02_10-04-24.png){: .align-left}  

I didnt want use metasploit module for this, so i quickly grabbed the exploit available from exploit database.
Had weird issues running the code, due to spacing i belive (DOS-style CRLF line endings), so i quickly converted the script using dostounix tool.
The exploit is a very simple command injection through a ping functionnality within the application.

![source-01](/img/Screenshot_2020-05-02_10-29-57.png){: .align-left}  

Running the exploit give us a shell as www-data 

![source-01](/img/Screenshot_2020-05-02_10-34-54.png){: .align-left}  

After some browsing around, I got to the database settings file which contained a password.

![source-01](/img/Screenshot_2020-05-02_10-42-54.png){: .align-left}  
	
Now lets quickly grab users from the system, we might be able to use the found credentials with the users

$ cat /etc/passwd | awk -F : '{print $1}'
root
jimmy
mysql
joanna

After trying this password on jimmy’s account, I was able to login to the box as Jimmy, classic case of password reuse!

![source-01](/img/Screenshot_2020-05-02_14-13-04.png){: .align-left}  

And now we have a shell a jimmy user. I was excepting to get user.txt flag from here, but no we need to enumerate further more to get anywhere.

jimmy@openadmin:/var/www$ ls -la
total 16
drwxr-xr-x  4 root     root     4096 Nov 22 18:15 .
drwxr-xr-x 14 root     root     4096 Nov 21 14:08 ..
drwxr-xr-x  6 www-data www-data 4096 Nov 22 15:59 html
drwxrwx---  2 jimmy    internal 4096 Nov 23 17:43 internal
lrwxrwxrwx  1 www-data www-data   12 Nov 21 16:07 ona -> /opt/ona/www

Found, interesting directory under /var/wwww which belongs to jimmy user.And an interesting php file which reads joana's SSH privatekeys.

![source-01](/img/Screenshot_2020-05-02_14-25-02.png){: .align-left}  

Time to do a curl request to that file and see if I can obtain the key file.


Lets chek listening ports

![source-01](/img/Screenshot_2020-05-02_14-35-51.png){: .align-left}  

i used curl to download the content of main.php

![source-01](/img/Screenshot_2020-05-02_14-47-04.png){: .align-left}  
And here we got an encrypted RSA private key, to make this usefull we need to crack the key.


