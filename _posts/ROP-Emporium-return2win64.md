## Introduction
I want get into pwn and learn more about ROP.I solved some basic challenges of ROP Emporium to improve my pwn and ROP knowledge. These challenges use the usual CTF objective of retrieving the contents of a file named "flag.txt" from a remote machine by exploiting a given binary.ROP theory is out of scope of this article, If you want ROP theory there are lot of good resource around.

## Description 
Locate a method within the binary that you want to call and do so by overwriting a saved return address on the stack.
Click below to download the binary. 

Lets check the security of the binary 
[+] checksec for '/root/Desktop/ROP/ret2win/ret2win'
Canary                        : ✘ 
NX                            : ✓ 
PIE                           : ✘ 
Fortify                       : ✘ 
RelRO                         : Partial

Because PIE isn't enabled the binary will be loaded at a fixed location into memory (0x400000).





