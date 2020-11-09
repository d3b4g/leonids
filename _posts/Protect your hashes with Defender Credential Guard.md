###Introduction:

Defender Credential Guard increases the security by taking advantage of platform security features including, Secure Boot and virtualization. It uses virtualization-based security to isolate secrets so that only privileged system software can access them. Credential Guard prevents these attacks by protecting NT LAN Manager protocol (NTLM) password hashes and Kerberos Ticket Granting Tickets. 

### Attack
I have windows server 2012 domain controller without credential guard enabled. Using mimikatz i can query the credential stored in the LSA process to get the NTLM hash of the accounts.


Prevention:

To prevent mimikatz from retriving  hashes LSA process, Enable credential guard through group policy.

