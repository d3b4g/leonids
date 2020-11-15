#### Introduction:

DCSync attacks allow an attacker to impersonate a domain controller and request password hashes from other domain controllers.

#### How the DCSync Attack Works?

To perform a DCSync attack, We must have compromised a user with the below permissions.
+ Replicating Directory Changes All
+ Replicating Directory Changes

By default in activedirectory, members of the below groups have those permissions.
+ Administrators
+ Domain Admins
+ Enterprise Admins 


#### Attack
For this scinario i have configured a user with below permission.
+ Replicating Directory Changes All
+ Replicating Directory Changes

The user PTRACE@CONTOSO.LOCAL has the DS-Replication-Get-Changes privilege on the domain CONTOSO.LOCAL

![source-01](/img/dcsyn1.PNG){: .align-left}

We can use secretdump for Dcsync

![source-01](/img/dcsyn3.PNG){: .align-left}

#### Detection:

There are someways to detect DCsync attack.

+ Event ID 4662 Monitoring

![source-01](/img/dcsyn4.PNG){: .align-left}

+ Network Monitoring1 - DsGeNCChange

#### Defense 
