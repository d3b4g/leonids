#### Monitoring PowerShell in the Enterprise 

Powershell is an extremely powerful scripting and administration language that is baked right into Windows, this make an attractive target for attackers. Over the years
powerShell is increasingly being used as an offensive tool for attacks by threat actors. There are a number of powershell-base offensive tools available, including Empire, PowerSploit, PSAttack and Nishang.

PowerShell supports three types of logging: 

+ Module logging
+ Script block logging
+ Transcription


#### Setting up PowerShell Logging
Logging must be configured through Group Policy as follows:

Administrative Templates → Windows Components → Windows PowerShell



#### Module Logging

Module logging records pipeline execution details as PowerShell executes.

+ Enable module logging:

#### Script Block Logging
This records all lines of code as they are executed by PowerShell.
