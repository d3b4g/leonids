#### Introduction

Last month, Microsoft patched a very interesting vulnerability (CVE-2020-1472), a privillege escalation vulnerability in Netlogon Remote Protocol that could allow anyone to take full control of windows domain controllers. This vulnerability has a huge impact as it basically allows any unauthenticated user on the local network to compromised the windows domain controller easily. The ZeroLogin vulnerability takes advantage of improper handling of the AES-CFB8 encryption cipher used by ComputeNetlogonCredential function. 
Security firm Secura published a detailed whitepaper about the vulnerability https://www.secura.com/pathtoimg.php?id=2055

#### Proof-of-Concept:

In last week several poc's for this vulnerability are surfaced




#### Affected Products
All Microsoft Windows Servers that use MS-NRPC to connect to a domain controller except server 2008 are affected.

Microsoft Windows Server 2019
Microsoft Windows Server 2016
Microsoft Windows Server 2012
Microsoft Windows Server 2012 R2
Microsoft Windows Server 2008 R2 Service Pack 1


#### Solution
Microsoft has released a security fix in its monthly Patch Tuesday updates for August 2020.
