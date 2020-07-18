---
layout: post
title:  "RopMev2 - Challenge"
date:   2020-07-17 14:07:20
categories: [ROP]
excerpt: "This is a recently retired pwn challenge from hackthebox."

comments: true
---


#### Summary

This is a recently retired pwn challenge from hackthebox. I will be using Cutter for reverse engineering this binary.

> Cutter is a Free and Open Source RE Platform powered by radare2


# Binary Analysis

Open the binary using Cutter and Select the **aaa** option to analyze the binary 

![source-01](/img/Screenshotr6.png){: .align-left}

aaa command  execute other below commands to analyze the binary.

    + aac - analyze function calls 
    + aar - analyze len bytes of instructions for references
    + afta - do type matching analysis for all functions


#### Binary protection :
Using checksec command we can check, the protection enabled in the binary.

![source-01](/img/ropv2-8.PNG){: .align-left}


#### Note(s):

+ As expected Binary has a non executable stack.


For futher analysis i decompile the binary, here is snippet of main's pseudocode.

![source-01](/img/ropv2-1.PNG){: .align-left}

#### Note(s):


+ Prints a welcome message "Please dont hack me"
+ Call to read that reads 500 bytes from stdin.
+ strcmp compares the entered string with "DEBUG"

From the main function graph view we can see, at the end the program it CALL's to another function.


![source-01](/img/ropemev2-001.PNG){: .align-left}

Lets have a look at this function.

> To decompile a function right-click and select show-in decompiler. 

![source-01](/img/ropv2-2.PNG){: .align-left}

#### Note(s):

What is the purpose of a function?

+ This function just apply RO13 to the inputs we enter 
+ It add or sub 0xd to every character we enter.
+ we can bypass this by adding a null byte to the begning of our payload

> ROT13 ("rotate by 13 places", sometimes hyphenated ROT-13) is a simple letter substitution cipher that replaces a letter with the 13th letter after it, in the alphabet.

# Exploitation

So now binary analysis out of the way lets start exploiting. 

I will use a pwntool based script for fuzzing the binary.

![source-01](/img/ropv2-10.PNG){: .align-left}


After running the script, monitor from the GDB for crashes.

![source-01](/img/ropv2-9.PNG){: .align-left}

#### Notes(s):

+ The program crashed and registers are overwritten, but thats not the payload we sent ! 
+ we send "A"s but it is overwritten with "nnnnn" 
+ So here what happend was the rot13 function messed up our payload. From the binary analysis we know there is function which apply ROT13 operation on the input we enter.


Lets modify our script to get around with rot13.

![source-01](/img/ropv2-7.PNG){: .align-left}

#### Note(s):

+ Open the program in GDB and set a break point
+ Send the unique pattern to the program
+ Added a nullbyte to the start of the payload to stop rot13 messing with it

After running the script, **PRESS** "c" to continue the execution.

Program received segmentation fault and crashed.

![source-01](/img/ropv2-4.PNG){: .align-left}

To calculate the offset copy and past the value in RIP to Metasploit pattern_offset tool.

![source-01](/img/ropv2-6.PNG){: .align-left}

#### Collecting ROP Gadgets

I used ROPGadget tool to find the gadgets.

![source-01](/img/ropv2-11.PNG){: .align-left}

Below are the gadgets we need to form our ropchain

![source-01](/img/ropv2-12.PNG){: .align-left}

Thats all we need to build the exploit. 

#### Final Exploit

Running the exploit, we get a local shell.

![source-01](/img/ropv2-13.PNG){: .align-left}


#### Conclusion:

This was a great challenge i learned few new techniiques.

> You can find all code for this challenge from my github repo: https://github.com/d3b4g/ropchallenges/







