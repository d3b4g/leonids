#### Monitoring PowerShell in the Enterprise 

Powershell is an extremely powerful scripting and administration language that is baked right into Windows, this make an attractive target for attackers. Over the years
PowerShell is increasingly being used as an offensive tool for attacks by threat actors. There are a number of powershell-based offensive tools used by security professionals, including Empire, PowerSploit, PSAttack and Nishang. 

Over the years microsoft implemented several security features into PowerShell, to help the defenders to detect and mitigate PowerShell based attacks. In PowerShell version 5.1 Microsoft introduced several security features.

+ AMSI (Antimalware Integration)
+ JEA
+ System-wide transcription
+ Constrained PowerShell
+ Protected Event Logging

#### PowerShell supports three types of logging: 

+ Module logging
+ Script block logging
+ Transcription


#### Setting up PowerShell Logging

This guide aims to help you configure Powershell logging for monitoring powershell abuse.


#### Module Logging

Module logging records pipeline execution details as PowerShell executes. It writes Event ID 4103 events to the Microsoft-Windows-PowerShell/Operational channel.

+ Enable module logging:

Computer Configuration › Administrative Templates › Windows Components › Windows PowerShell and open the Turn on PowerShell Module Logging setting

![source-01](/img/powershell2.PNG){: .align-left}

#### Script Block Logging

Script Block Logs contents of all the script blocks processed by the PowerShell engine. It writes Events ID 4104 to the Microsoft-Windows-PowerShell/Operational channel.


+ Enable Script Block Logging:

Computer Configuration › Administrative Templates › Windows Components › Windows PowerShell and open the Turn on PowerShell Script Block Logging setting

![source-01](/img/powershell3.PNG){: .align-left}

#### Transcription Logging

By enabling transcription logging it creates a unique record of every PowerShell session, including all input and output. Transcriptions are written to the current user’s Documents directory unless an output directory is not defined in the policy setting.


+ Enable Transcription Logging:

Go to Computer Configuration › Administrative Templates › Windows Components › Windows PowerShell and open the Turn on PowerShell Transcription setting.

![source-01](/img/powershell1.PNG){: .align-left}

The transcriptions are written as text files and can grow in size quickly. 


#### Conclusion

PowerShell generate large number of events and logs, due to this, organizations should carefully consider which events to log and send to log Log Aggregator or SIEM. 
