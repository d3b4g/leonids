Active Directory Red Team Domain Enumeration

You just cant throw exploit here and there and expect to work. Enumeration is the key for any successful engagement, in this module i will cover how you can enumerate microsoft Active Directory with Powerview and native windows functionalities. Powerview includes many commands to enumerate and manage the active directory. I will demostrate the commands i mostly use in Active Directory assessments.

In this module im going to cover the following.

-    Enumerating Domain
-    Enumerating AD Users
-    Enumerating AD Groups
-    Enumerating AD Computers
-    Enumerating Domain ACLs with built in tools  and Powerview
-    Enumerating Group Policy Objects (GPOs)
-    Enumerating AD Trusts

##### Get-NetDomain 

This command will give information about the current domain domain controller.

![source-01](/img/enu1.PNG){: .align-left}

-Domain parameter in Get-Net-Domain cmdlt allow you to specify domain name you want enumerate

![source-01](/img/enu2.PNG){: .align-left}

#### Get-NetUser

This cmdlt enumerates the users and dump usefull informations about the user

![source-01](/img/enu3.PNG){: .align-left}


#### Get-Net User

The output of Get-Net User cmdlt is bit messy, so you can pipe Get-Net User cmdlt and select specific objects you want output.

![source-01](/img/enu4.PNG){: .align-left}

#### Get-NetGroupMembers

This cmdlt allow you to enumerate group emembership from active directory, this is very usefull in an engagement.

![source-01](/img/enu5.PNG){: .align-left}
