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

from there when we login it redirect to /ona
![source-01](/img/Screenshot_2020-05-02_09-40-35.png){: .align-left}  
