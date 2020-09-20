#### Introduction

Last month, Microsoft patched a very interesting vulnerability (CVE-2020-1472), a privillege escalation vulnerability in Netlogon Remote Protocol that could allow anyone to take full control of windows domain controllers. This vulnerability has a huge impact as it basically allows any unauthenticated user on the local network to compromised the windows domain controller easily. The ZeroLogin vulnerability takes advantage of improper handling of the AES-CFB8 encryption cipher used by ComputeNetlogonCredential function. 
Security firm Secura published a detailed whitepaper about the vulnerability https://www.secura.com/pathtoimg.php?id=2055

Proof of Concept Demo




