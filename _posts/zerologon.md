#### Introduction

Last month, Microsoft patched a very interesting vulnerability (CVE-2020-1472), a privillege escalation vulnerability in Netlogon Remote Protocol that could allow anyone to take full control of windows domain controllers. This vulnerability has a huge impact as it basically allows any unauthenticated user on the local network to compromised the windows domain controller without having any special privilledges.  

#### Vulnerability detail

The ZeroLogin vulnerability takes advantage of improper handling of the AES-CFB8 encryption cipher used by ComputeNetlogonCredential function. An attacker could exploit this vulnerability to spoof the identity of any machine on a network when attempting to authenticate to the Domain Controller.

Security firm Secura published technical detailes about the vulnerability https://www.secura.com/blog/zero-logon



#### Proof-of-Concept:

In last week several poc's for this vulnerability are surfaced. i ended up using this poc from github for this demostration. My target is a recently retired hackthebox machine.

Run the exploit against target:

![source-01](/img/zero1.PNG){: .align-left}


Exploit completed successfully, now we can dump the NTDS database using secretdump tool

![source-01](/img/zero2.PNG){: .align-left}

Finally can use these hashes for pass-the-hash or cracking. As you can see this vulnerability is so simple to exploit and unauthenticated attacker is able to obtain full administrator privileges on Active Directory within few minutes. Please do not run the exploit in a production enviornment if you don't know how to revert back the changes.


#### Affected Products

All Microsoft Windows Servers that use MS-NRPC to connect to a domain controller except server 2008 are affected.

+ Microsoft Windows Server 2019
+ Microsoft Windows Server 2016
+ Microsoft Windows Server 2012
+ Microsoft Windows Server 2012 R2
+ Microsoft Windows Server 2008 R2 Service Pack 1


#### Mitigation Actions for Zerologon

The Zerologon vulnerability has been patched by Microsoft with the August 2020 security updates.
