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

Module logging records pipeline execution details as PowerShell executes. This feature writes Event ID 4103 events to the Microsoft-Windows-PowerShell/Operational channel. This can be enabled by setting the LogPipelineExecutionDetails property of a module to True. Or through Group Policy

+ Enable module logging:


![source-01](/img/powershell2.PNG){: .align-left}

#### Script Block Logging

This records all lines of code as they are executed by PowerShell. Events with ID 4104 are written to the Microsoft-Windows-PowerShell/Operational channel.


+ Enable Script Block Logging:

Computer Configuration › Administrative Templates › Windows Components › Windows PowerShell and open the Turn on PowerShell Script Block Logging setting

![source-01](/img/powershell3.PNG){: .align-left}

#### Transcription Logging

Go to Computer Configuration › Administrative Templates › Windows Components › Windows PowerShell and open the Turn on PowerShell Transcription setting.

![source-01](/img/powershell1.PNG){: .align-left}

