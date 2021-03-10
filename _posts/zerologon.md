Last week, Microsoft release out of band updates for critical Exchange server vulnerabilities which affect on-premises version of exchange server.(CVE-2020-1472)

Vulnerability detail
The ZeroLogin vulnerability takes advantage of improper handling of the AES-CFB8 encryption cipher used by ComputeNetlogonCredential function. An attacker could exploit this vulnerability to spoof the identity of any machine on a network when attempting to authenticate to the Domain Controller.

Security firm Secura published technical detailes about the vulnerability https://www.secura.com/blog/zero-logon

Proof-of-Concept demo:
