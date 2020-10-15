#### Monitoring PowerShell in the Enterprise 

Powershell is an extremely powerful scripting and administration language that is baked right into Windows, this make an attractive target for attackers. Over the years
powerShell is increasingly being used as an offensive tool for attacks by threat actors. There are a number of powershell-base offensive tools used by security professionals, including Empire, PowerSploit, PSAttack and Nishang.

In PowerShell version 5.1 Microsoft implemented several security features.
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
Logging can be configured through Group Policy as follows:

Administrative Templates → Windows Components → Windows PowerShell



#### Module Logging

Module logging records pipeline execution details as PowerShell executes.

+ Enable module logging:

#### Script Block Logging
This records all lines of code as they are executed by PowerShell.
