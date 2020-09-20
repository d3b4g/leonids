#### Introduction

Last month, Microsoft patched a very interesting vulnerability (CVE-2020-1472), a privillege escalation vulnerability in Netlogon that could grant anyone full takeover of domain controllers.This attack has a huge impact as it basically allows any unauthenticated user on the local network to compromised the windows domain controller easily. This vulnerability takes advantage of a security flaw in a cryptographic authentication scheme (AES-CFB8) it use.


