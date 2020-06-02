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

> Ganache CLI, part of the Truffle suite of Ethereum development tools, is the command line version of Ganache, your personal blockchain for Ethereum development.

Ganache CLI uses ethereumjs to simulate full client behavior and make developing Ethereum applications faster, easier, and safer. It also includes all popular RPC functions and features (like events) and can be run deterministically to make development a breez.

To interact with the Ethereum blockchain, i will be using python library web3.py. To interact with the Smart Contract, we need:

+ The address of the contract 
+ Application Binary Interface of the contract

To test this vulnerability we can use setDomain() function and set the domain to our localhost  and then call getDomain(). This will make the contract ping localhost, if ping successful we know we got remote code execution.

###### Building Python Script
So let’s start building a python script to interact with this contract.
```python

from web3 import Web3
import json
```
+ web3.Web3 intract with the blockchain
+ json to load the WeaponizedPing.json





