### Introduction:

Defender Credential Guard increases the security by taking advantage of platform security features including, Secure Boot and virtualization. It uses virtualization-based security to isolate secrets so that only privileged system software can access them. Credential Guard prevents these attacks by protecting NT LAN Manager protocol (NTLM) password hashes and Kerberos Ticket Granting Tickets. 

### Attack
I have windows server 2012 domain controller without credential guard enabled. Using mimikatz i can query the credential stored in the LSA process to get the NTLM hash of the accounts.

![source-01](/img/mimikatz1.PNG){: .align-left}

Mimikatz was able to retrive NTLM hashes from the LSA process, attacker can use these hashes for lateral movement.


### Prevention:

To prevent mimikatz from retriving  hashes LSA process, Enable credential guard through group policy.

To enabled the device guard: 
> From the Group Policy Management Console go to Computer Configuration -> Administrative Templates -> System -> Device Guard.
  Double-click Turn On Virtualization Based Security, and then click the Enabled option.

![source-01](/img/mimikatz2.PNG){: .align-left}

Device Guard group policy gives you the option of "Secure Boot with DMA Protection".
