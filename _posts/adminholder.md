####  AdminSDHolder Attacks and defence

#### What is an AdminSDHolder?

AdminSDHolder is an object in Active Directory that acts as a security descriptor template for protected accounts and groups in an Active Directory domain.
Active Directory Domain Services uses AdminSDHolder, protected groups and Security Descriptor propagator (SD propagator or SDPROP for short) to secure privileged users and groups from unintentional modification. 
AdminSDHolder object SDProp runs by default every sixty minutes and reset the permissions on the protected accounts to match those of the AdminSDHolder container.
