**Carrier** Chainsaw is a retired vulnerable VM from Hack. This box is about smart contract exploitation.

#### Enumeration

To begin the enumeration process, a **TCP/UDP** port scan was run against the target using nmap. The purpose of scan is to quickly determine which ports are open and which services are running. 

###### OS Information
Based on the Apache version, it looks like we are dealing with Xenial / Ubuntu 16.04 based system.

###### Open Ports and Services :
In addition to usual ports

+ Port 21 : FTP Server which allows anonymous login
+ Port 22 : SSH Server
+ Port 9810 : A webserver ?

Quickly checking VPN reveals some interesting files, downloaded all the files to local machine.
```python

➜  chainsaw ftp 10.10.10.142
Connected to 10.10.10.142.
220 (vsFTPd 3.0.3)
Name (10.10.10.142:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001        23828 Dec 05  2018 WeaponizedPing.json
-rw-r--r--    1 1001     1001          243 Dec 12  2018 WeaponizedPing.sol
-rw-r--r--    1 1001     1001           44 May 31 16:20 address.txt
226 Directory send OK.
ftp> mget *
```
We have got three interesting files from FTP server, after a bit of researching i found out these are components of a smart contract

WeaponizedPing.json - A compiled Solidity smart-contract in JSON format
WeaponizedPing.sol - An Ethereum smart-sontract written in Solidity
address.txt - Possibly an address on a blockchain where the contract is been deployed.


