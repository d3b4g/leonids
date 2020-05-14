---
layout: post
title:  "ROP Emporium - split"
date:   2020-05-8 14:07:19
categories: [ROP]
excerpt: "Doing these challenges to improve my binary exploitation skills and teach my self Return oriented programming (ROP). These challenges use the usual CTF objective of retrieving the contents of a file named flag.txt from a remote machine by exploiting a given binary"

comments: true
---


## Introduction
This is the Second challenge from ROP Emporium named "Split". In this i am, going to write a small ROP chain to complete this challenge. Going to use radare2 as much as i can, just for the sake of learning the tool. Radare2 is a complete framework for reverse-engineering and analyzing binaries.

###### Challenge Description 
Combine elements from the ret2win challenge that have been split apart to beat this challenge. Learn how to use another tool whilst crafting a short ROP chain. 

###### About the Binary:
Our binary is usual ELF executable in 64-bit architecture.Lets check what protection are on this binary, using rabin2, which comes with radare2 framework.

![source-01](/img/Screenshot_2020-05-13_12-32-28.png){: .align-left}


PIE isn't enabled and nx set to true, we know shellcode cannot be executed off the stack and we know binary has ASLR disabled.

## Analyzing the 64bit ELF binary
Lets load the binary with radare2 and dump the functions.The afl command list the functions. We can also output it as JSON using this command aflj~{}

![source-01](/img/Screenshot_2020-05-13_08-41-16.png){: .align-left}


Here we can see interesting functions:
- main()
- pwnme()
- usefulFunction()

So lets disassemble and see what these fucntions does!

###### main():


![source-01](/img/Screenshot_2020-05-13_08-44-57.png){: .align-left}


The only interesting thing here for us is, its calling the function name pwnme() which have overflow vulnerability.

###### pwnme():


![source-01](/img/Screenshot_2020-05-13_08-48-45.png){: .align-left}


Just like in retwin challenge, we have a 32 byte buffer that can be overflowed with fgets that takes 96 characters from the user.


###### usefulFunction()


![source-01](/img/Screenshot_2020-05-13_08-34-01.png){: .align-left}


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

#### Building ROP-Chain
Now we know what we need to build our ROP syscall: 
(1) A string containing “/bin/cat flag.txt”, 
(2) the address of system and 
(3) a gadget to add “/bin/cat flag.txt” to rdi register, pop rdi; ret, is what we want.

let's find the pop rdi; ret; so the string would be provided to system for execution.


![source-01](/img/Screenshot_2020-05-14_08-23-29.png	){: .align-left}



w00t we got the flag!

###### End 

Thats the end of ret2win challenge,it is a very simple one but as i progress through the challenges the exploits will become more complex and fun. Remember never stop learning, Cheers!.

> Code for this challenge  https://github.com/d3b4g/ROP-Emporium



