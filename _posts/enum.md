Active Directory Red Team Domain Enumeration

You just cant throw exploit here and there and expect to work. Enumeration is the key for any successful engagement, in this module i will cover how you can enumerate microsoft Active Directory with Powerview and native windows functionalities. Powerview includes many commands to enumerate and manage the active directory. I will demostrate the commands i mostly use in Active Directory assessments. 
> If you have not read my previous post about RedTeam Lab setup, please read

In this module im going to cover the following.

-    Enumerating Domain
-    Enumerating AD Users
-    Enumerating AD Groups
-    Enumerating AD Computers
-    Enumerating Domain ACLs with built in tools  and Powerview
-    Enumerating Group Policy Objects (GPOs)
-    Enumerating AD Trusts
-    Enumerating Local Administrators

##### Enumerating Domain

##### Description 
In Active Directory terms, a domain is an area of a network organized by a single authentication database. In other words, an Active Directory domain is essentially a logical grouping of objects on a network. Domains are created so IT teams can establish administrative boundaries between different network entities. In Active Directory domains are controlled by a tool called the domain controller.

- Get-NetDomain 

This command will give information about the current domain domain controller.

![source-01](/img/enu1.PNG){: .align-left}

-Domain parameter in Get-Net-Domain cmdlt allow you to specify domain name you want enumerate

![source-01](/img/enu2.PNG){: .align-left}

###### Enumerating AD Users

##### Description
A user object in AD is used to represent a real user in an organizational network environment.

 - Get-NetUser
This cmdlt enumerates the users and dump usefull informations about the user

![source-01](/img/enu3.PNG){: .align-left}

The output of Get-Net User cmdlt is bit messy, so you can pipe Get-Net User cmdlt and select specific objects you want output.

![source-01](/img/enu4.PNG){: .align-left}

Other useful properties i always look for are
- pwdlastset
- badpasswordcount


##### Enumerating AD Groups

##### Description
The Active Directory groups are a collection of Active Directory objects. The group can include users, computers, other groups, and other AD objects.

- Get-NetGroupMembers

This cmdlt allow you to enumerate group emembership from active directory, this is very usefull in an engagement.

![source-01](/img/enu5.PNG){: .align-left}

#### Enumerating AD GPO

##### Description
Group Policy Objects are Active Directory containers used to store groupings of policy settings. These objects are then linked to specific sites, domains, or most commonly specific organizational units (OUs).

With PowerView, the **Get-NetGPO**  cmdlet allows for the easy enumeration of all current GPOs in a given domain.

![source-01](/img/enu6.PNG){: .align-left}

##### Enumerating AD Computers
##### Description
Computer objects are used to uniquely identify and manage Windows-based domain clients within Active Directory. They are used to specify computer names, locations, properties and access rights.

![source-01](/img/enu8.PNG){: .align-left}

As you can see **Get-NetComputer** cmdlt gives a lot of information about the computer object. You always don't need all the information, you can always use the handy pipe and select the information you need.

![source-01](/img/enu9.PNG){: .align-left}

##### Enumerating Domain ACLs 
##### Description:
ACLs (Access Control Lists) are the settings that define what objects get access to other objects in Active Directory. There are two types of ACLs:
- Discretionary access control list (DACL): This defines the security principals which are either granted or denied access to a securable object.
- System access control list (SACL): This grants power to administrators to log the access attempts made to secured objects.

![source-01](/img/enu10.PNG){: .align-left}

![source-01](/img/enu15.PNG){: .align-left}

This cmdlt outputs the list of ACEs applied to the object. 

 #### Enumerating AD Trusts
 ##### Description:
Active Directory domain to domain communications occur through a trust. An AD DS trust is a secured, authentication communication channel between entities, such as AD DS domains, forests. Trusts enable you to grant access to resources to users, groups and computers across entities.

**Trusts Direction:**
- Two-way trust (Bi-directional): Users from Domain A can access resources in Domain B
and vice versa.
- One-way trust (Unidirectional): Users in the trusted domain can access resources in the
trusting domain but the reverse is not true
**Trusts Transitivity:**
- Parent-child trust: It is created automatically between the new domain and the domain
that precedes it in the namespace hierarchy, whenever a new domain is added in a tree.
For example, usa.fanzy.com is a child of fanzy.com). This trust is always two-way
transitive.
- Tree-root trust: It is created automatically between whenever a new domain tree is
added to a forest root. This trust is always two-way transitive.
**External Trusts:** Between two domains in different forests when forests do not have a trust
relationship. It can be one-way or two-way and is nontransitive.

- Get-NetDomainTrust 
This cmdlt get all domain trusts including parent, childand external)

![source-01](/img/enu11.PNG){: .align-left}

- Get-NetForestDomain | Get-NetDomainTrust

Piping the result of **Get-NetforestDomain** to **Get-NetDomainTrust**  enumerate all the trusts of all the domains found 

![source-01](/img/enu12.PNG){: .align-left}

We can also enumerate this information with .NET class "Domain.GetAllTrustRelationships"

![source-01](/img/enu14.PNG){: .align-left}

#### Get-DomainPolicy 

![source-01](/img/enu13.PNG){: .align-left}


This cmdtlt get info about the policy

(Get-DomainPolicy)."KerberosPolicy" 
This cmdlt gives Kerberos tickets info(MaxServiceAge)

(Get-DomainPolicy)."SystemAccess" 

This cmdlt gives Password policy

####  Enumerating Local Administrators
##### Desciption
Get local group membership with the NetLocalGroupGetMembers Win32 API call.

