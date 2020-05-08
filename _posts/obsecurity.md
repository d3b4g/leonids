---
layout: post
title:  "Hackthebox - Obsecurity"
date:   
categories: [Hackthebox]
excerpt: "Obsecurity is rated as a Medium Linux box. It was released on 04 Jan 2020 and has been created by @dmw0ng

This box required us to perform the following tasks:


- Enumerate a web server to find vulnerable web application

##nmap
➜  Obsecurity nmap -sC -sV 10.10.10.168 -vvv

Initiating NSE at 07:08
Completed NSE at 07:08, 0.00s elapsed
Nmap scan report for 10.10.10.168
Host is up, received reset ttl 63 (0.35s latency).
Scanned at 2020-05-08 07:07:26 EDT for 72s
Not shown: 996 filtered ports
Reason: 996 no-responses
PORT     STATE  SERVICE    REASON         VERSION
22/tcp   open   ssh        syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 33:d3:9a:0d:97:2c:54:20:e1:b0:17:34:f4:ca:70:1b (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDMGbaGqnhJ5GoiEH4opql41JoOBmB089aA2wQZgp5GCxf3scJti3BWS30ugrj2PBMydulKmeiAbHWA37ojLyAJxSdvyWrPqneEZfdaMCm/9NPnPSouZgQKLoOg/w8DEPeXfon8bxGYOt3HMXtVMk04/kt09ad7E2Eej8WzAp2k3JJX17ndZL0S5UNDJFyh6pHhGqCtjOapLGb1QwS7BDw+kHiZrkZbDRa1rMv5a/QoljgOIq0byvm5jEVe4NhKKfgwH7kXEU1DAlXmWYzsq/ZdhhwutrjbDam5alw4UAE/35DcPlnVl/7eRK6RIARJPZEQ0O64ixlzbAfIcDGi8GOr
|   256 f6:8b:d5:73:97:be:52:cb:12:ea:8b:02:7c:34:a3:d7 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAygZOWHjNzQWySvfTX7s9Cnz0eSrc9IS/8wk126Wby5EAUmSalXlAL5WETz8nu/JN8nVpgHYEW6/mZm071xMd0=
|   256 e8:df:55:78:76:85:4b:7b:dc:70:6a:fc:40:cc:ac:9b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ6lPjlgfOScC0NXPX926fST+MXZViZJzBQPXDWsdHuw
80/tcp   closed http       reset ttl 63
8080/tcp open   http-proxy syn-ack ttl 63 BadHTTPServer
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Fri, 08 May 2020 11:08:50
|     Server: BadHTTPServer
|     Last-Modified: Fri, 08 May 2020 11:08:50
|     Content-Length: 4171
|     Content-Type: text/html
|     Connection: Closed
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>0bscura</title>
|     <meta http-equiv="X-UA-Compatible" content="IE=Edge">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta name="keywords" content="">
|     <meta name="description" content="">
|     <!-- 
|     Easy Profile Template
|     http://www.templatemo.com/tm-467-easy-profile
|     <!-- stylesheet css -->
|     <link rel="stylesheet" href="css/bootstrap.min.css">
|     <link rel="stylesheet" href="css/font-awesome.min.css">
|     <link rel="stylesheet" href="css/templatemo-blue.css">
|     </head>
|     <body data-spy="scroll" data-target=".navbar-collapse">
|     <!-- preloader section -->
|     <!--
|     <div class="preloader">
|     <div class="sk-spinner sk-spinner-wordpress">
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Fri, 08 May 2020 11:08:51
|     Server: BadHTTPServer
|     Last-Modified: Fri, 08 May 2020 11:08:51
|     Content-Length: 4171
|     Content-Type: text/html
|     Connection: Closed
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>0bscura</title>
|     <meta http-equiv="X-UA-Compatible" content="IE=Edge">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta name="keywords" content="">
|     <meta name="description" content="">
|     <!-- 
|     Easy Profile Template
|     http://www.templatemo.com/tm-467-easy-profile
|     <!-- stylesheet css -->
|     <link rel="stylesheet" href="css/bootstrap.min.css">
|     <link rel="stylesheet" href="css/font-awesome.min.css">
|     <link rel="stylesheet" href="css/templatemo-blue.css">
|     </head>
|     <body data-spy="scroll" data-target=".navbar-collapse">
|     <!-- preloader section -->
|     <!--
|     <div class="preloader">
|_    <div class="sk-spinner sk-spinner-wordpress">
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: BadHTTPServer
|_http-title: 0bscura
9000/tcp closed cslistener reset ttl 63
1


