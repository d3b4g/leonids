
#### Enumeration 

To begin the enumeration process, a TCP/UDP port scan was run against the target using nmap. The purpose of scan is to quickly determine which ports are open and which services are running. 

##### OS Information:
Based on the Apache version, it looks like we are dealing with Xenial / Ubuntu 16.04 based system.

###### Open Ports:
In addition to usual ports, 22 and 80 UDP port 161 is open. UDP 161 is the standard SNMP port.There is also a filtered ftp port showing.

```python
➜  carrier gobuster dir -w /usr/share/wordlists/dirb/common.txt -u 10.10.10.105 -t20 

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.105
[+] Threads:        20
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/05/27 16:12:03 Starting gobuster
===============================================================
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/.hta (Status: 403)
/css (Status: 301)
/debug (Status: 301)
/doc (Status: 301)
/fonts (Status: 301)
/img (Status: 301)
/index.php (Status: 200)
/js (Status: 301)
/server-status (Status: 403)
/tools (Status: 301)
===============================================================
2020/05/27 16:13:25 Finished
===============================================================


```
