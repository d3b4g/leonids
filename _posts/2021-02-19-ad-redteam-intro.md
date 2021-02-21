---
layout: post
title:  "Active Directory Red Team - Lab Setup"
date:   2021-02-19 14:07:20
categories: [Active Directoru Red Team]
excerpt: "Enterprises are using Microsoft Active Directory for identity management and protecting resources. As a blue or red teamer knowing common Active Directory security issues and how to secure them is very important"

comments: true
---



Enterprises are using Microsoft Active Directory for identity management and protecting resources. As a blue or red teamer knowing common Active Directory security issues and how to secure them is very important.

### What you will learn

In this series of posts, you will be guided through red team oriented Active Directory attacks, exploiting common misconfigurations and abusing Active Directory functionality to assess Active Directory security.

##### Some of the attack scenarios i would like to cover:

+ Active Directory Enumeration
+ Active Directory Enumeration with Bloodhound
+ Abusing Active Directory Object Permissions ACLs / ACE
+ Kerberos Attacks
+ SMB Relay Attacks
+ Domain Persistence
+ Credential Attacks
+ Kerberos Delegeation Attacks
   + Unconstrained Delegation
   + Constrained Delegation
   + Resource based Constrained Delegeation
+ Attacking Active Directory Forest
+ Abusing Microsoft LAPS
+ Abusing SQL Server

### Lab Topology
![source-01](/img/labssz.png)


I have built a red team cyber range that mimics real-world Active Directory environment  for demostrating the attacks. If you are looking to setup similar lab environment there are plenty of resource available, i am not going to cover lab setup in this post.

In the lab setup i have three domains.

+ Forest Root 
+ External Trusted Domain
+ Child Domain
 

#### Hardware Specifications:
+ Processor: i7-7700
+ RAM: 32GB RAM
+ HDD: 1TB  SSD
+ OS: Windows Server 2016, Windows Server 2012
+ Virtualization Software: VMware Workstation


#### Refference

