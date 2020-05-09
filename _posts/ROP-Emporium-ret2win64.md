
---
layout: post
title:  "ROP-Emporium-ret2win"
date:  
categories: [Return-oriented programming]
excerpt: "Recently while i was doing a Hackthebox machine which involves binary exploitation and ROP, i was really struggling to make things works."
---

## Introduction
Recently while i was doing a Hackthebox machine which involves binary exploitation and ROP, i was really struggling to make things works. so to improve my pwn and ROP skills i decide to do various ROP challenges, im starting with ROP Emporium challenges. These challenges use the usual CTF objective of retrieving the contents of a file named "flag.txt" from a remote machine by exploiting a given binary.
ROP theory is out of scope of this article, If you want ROP theory there are lot of good resources out there.

## Description 
Locate a method within the binary that you want to call and do so by overwriting a saved return address on the stack.
Click below to download the binary. 

### About the Binary:
Our binary is usual ELF executable in 64-bit architecture. 

![source-01](/img/Screenshot_2020-05-09_11-12-06.png){: .align-left}


PIE isn't enabled so the binary will be loaded at a fixed location into memory (0x400000) everytime.With nx set to true, we know shellcode cannot be executed off the stack and we know binary has ASLR disabled.

## Analyzing the 64bit ELF binary
Lets load the binary with GDB and dump the functions.
```
gef➤  info functions
All defined functions:

Non-debugging symbols:
0x00000000004005a0  _init
0x00000000004005d0  puts@plt
0x00000000004005e0  system@plt
0x00000000004005f0  printf@plt
0x0000000000400600  memset@plt
0x0000000000400610  __libc_start_main@plt
0x0000000000400620  fgets@plt
0x0000000000400630  setvbuf@plt
0x0000000000400640  __gmon_start__@plt
0x0000000000400650  _start
0x0000000000400680  deregister_tm_clones
0x00000000004006c0  register_tm_clones
0x0000000000400700  __do_global_dtors_aux
0x0000000000400720  frame_dummy
0x0000000000400746  main
0x00000000004007b5  pwnme
0x0000000000400811  ret2win
0x0000000000400840  __libc_csu_init
0x00000000004008b0  __libc_csu_fini
0x00000000004008b4  _fini
```
Here we can see interesting functions:
- main()
- pwnme()
- ret2win()

So lets see what this fucntions does!

main():
![source-01](/img/Screenshot_2020-05-09_11-15-15.png){: .align-left}





