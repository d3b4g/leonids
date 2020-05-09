
---
layout: post
title:  "ROP-Emporium-ret2win"
date:  
categories: [Return-oriented programming]
excerpt: "Recently while i was doing a Hackthebox machine which involves binary exploitation and ROP, i was really struggling to make things works."

## Introduction
Recently while i was doing a Hackthebox machine which involves binary exploitation and ROP, i was really struggling to make things works. so to improve my pwn and ROP skills i decide to do various ROP challenges, im starting with ROP Emporium challenges. These challenges use the usual CTF objective of retrieving the contents of a file named "flag.txt" from a remote machine by exploiting a given binary.
ROP theory is out of scope of this article, If you want ROP theory there are lot of good resources out there.

## Description 
Locate a method within the binary that you want to call and do so by overwriting a saved return address on the stack.
Click below to download the binary. 

About the Binary:
Our binary is usual ELF executable in 64-bit architecture. 
