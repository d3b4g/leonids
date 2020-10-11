####  AdminSDHolder Attack and Defence

#### What is an AdminSDHolder?

AdminSDHolder is an object in Active Directory that acts as a security descriptor template for protected accounts and groups in an Active Directory domain.
Active Directory Domain Services uses AdminSDHolder, protected groups and Security Descriptor propagator to secure privileged users and groups from unintentional modification. 
SDProp or security descriptor progagator runs by default every sixty minutes on  the domain controller holding the PDC emulator FSMO role and reset the permissions on the protected accounts to match those of the AdminSDHolder container.

The main function of SDPROP is to protect highly-privileged Active Directory accounts, ensuring that they can’t be deleted or have rights modified, accidentally or intentionally, by users or processes with less privilege.

![source-01](/img/0110.PNG){: .align-left}

#### AdminSDHolder Default Protected Objects

+ Account Operators
+ Administrator
+ Administrators
+ Backup Operators
+ Domain Admins
+ Domain Controllers
+ Enterprise Admins
+ Krbtgt
+ Print Operators
+ Read-only Domain Controllers
+ Replicator
+ Schema Admins
+ Server Operators

Using powershell we can view the users in AdminSDHolders container.

![source-01](/img/0112.PNG){: .align-left}


#### Attack

After compromising a domain controller, attackers use AdminSDHolder for perssistance by modifing Access Control List(ACL)




#### Defence

Domain Admins or higher privilledge users will be able to nodify the AdminSDHolder permions, protecting privillegedes yours prevents an attacker compromising or backdooring the AdminSDHolder. Monitoring and alerting any changes to AdminSDholder object permissions is very important. 
