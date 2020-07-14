#### Summary

This is a recently retired pwn challenge from hackthebox. I will be using Cutter for reverse engineering this binary

#### Binary's protection

Open the binary using Cutter

![source-01](/img/Screenshotr5.png){: .align-left}

Select the aaa option to analyze the binary 

![source-01](/img/Screenshotr6.png){: .align-left}

aaa command  execute other below commands to analyze the binary.

    + aa - alias for af@@ sym.*;af@entry0;afva
    + aac - analyze function calls (af @@ `pi len~call[1]`)
    + aar - analyze len bytes of instructions for references
    + aan - autoname functions that either start with fcn.* or sym.func.*
    + afta - do type matching analysis for all functions

#### Note(s):

+ Binary has a non executable stack

# Binary Analysis

I will use Cutter for binary analysis.

After decompile the binary, here is snippet of main's pseudocode:

![source-01](/img/ropv2-1.PNG){: .align-left}

#### Note(s):


+ Prints a welcome message "Please dont hack me"
+ Call to read that reads 500 bytes from stdin.
+ strcmp compares the entered string with "DEBUG"

From the main function graph view we can see, at the end the program CALL's to another function.


![source-01](/img/ropemev2-001.PNG){: .align-left}

Lets have a look at this function. I decompiled the function with Cutter. 

> To decompile a function from function view right-click and select show-in decompiler. 

![source-01](/img/ropv2-2.PNG){: .align-left}

#### Note(s):

+ This function just apply RO13 to the inputs we enter 
+ It add or sub 0xd to every character we enter.

> ROT13 ("rotate by 13 places", sometimes hyphenated ROT-13) is a simple letter substitution cipher that replaces a letter with the 13th letter after it, in the alphabet.

# Exploitation

So now binary analysis out of the way lets start exploiting. 

I will use below pwntool based script for fuzzing the binary.

![source-01](/img/ropv2-7.PNG){: .align-left}


Program received segmentation fault and crashed. Now from GDB we can

![source-01](/img/ropv2-4.PNG){: .align-left}

To calculate the offset copy and past the value is RSP to Metasploit pattern_offset tool.

![source-01](/img/ropv2-6.PNG){: .align-left}

#### Finding Gadgets

I used ROPGadget tool to find the gadgets.

pop_rdi_ret = 0x000000000040142b # pop rdi ; ret
syscall = 0x0000000000401168 # syscall
pop_rax = 0x0000000000401162 # pop rax ; ret
pop_rsi_r15 = 0x0000000000401429 # pop rsi ; pop r15 ; ret
pop_rdx_r13 = 0x0000000000401164 # pop rdx ; pop r13 ; ret








