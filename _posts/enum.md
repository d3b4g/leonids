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

##### Enumerating Domain
- Get-NetDomain 

This command will give information about the current domain domain controller.

![source-01](/img/enu1.PNG){: .align-left}

-Domain parameter in Get-Net-Domain cmdlt allow you to specify domain name you want enumerate

![source-01](/img/enu2.PNG){: .align-left}

###### Enumerating AD Users
 - Get-NetUser

This cmdlt enumerates the users and dump usefull informations about the user

![source-01](/img/enu3.PNG){: .align-left}


#### Get-Net User

The output of Get-Net User cmdlt is bit messy, so you can pipe Get-Net User cmdlt and select specific objects you want output.

![source-01](/img/enu4.PNG){: .align-left}

##### Enumerating AD Groups
##### Description
- Get-NetGroupMembers

This cmdlt allow you to enumerate group emembership from active directory, this is very usefull in an engagement.

![source-01](/img/enu5.PNG){: .align-left}

#### Enumerating AD GPO

##### Description
Group Policy Objects are Active Directory containers used to store groupings of policy settings. These objects are then linked to specific sites, domains, or most commonly specific organizational units (OUs).

With PowerView, the **Get-NetGPO**  cmdlet allows for the easy enumeration of all current GPOs in a given domain.

![source-01](/img/enu6.PNG){: .align-left}

##### Enumerating AD Computers
![source-01](/img/enu8.PNG){: .align-left}

enumerating
![source-01](/img/enu9.PNG){: .align-left}

##### Enumerating Domain ACLs with built in tools  and Powerview

![source-01](/img/enu10.PNG){: .align-left}

 #### Enumerating AD Trusts

Get-NetDomainTrust #Get all domain trusts (parent, children and external)
![source-01](/img/enu11.PNG){: .align-left}

Get-NetForestDomain | Get-NetDomainTrust #Enumerate all the trusts of all the domains found
![source-01](/img/enu12.PNG){: .align-left}
