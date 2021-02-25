Active Directory Red Team Domain Enumeration

Domain Enumeration is very crucial in any engagement, in this post i will cover how you can enumerate domain with Powerview and  .NET System.DirectoryServices.ActiveDirectory.Domain Class. Most of the time in real world engagement you don't always have permissions of running tools like powerview, in such case Active Directory .NET clasess are very handy.

Get-NetDomain 

This command will give information about the current domain domain controller.

![source-01](/img/enu1.PNG){: .align-left}

Get-NetDomain -Domain contoso.ext specify -Domain Parameter

![source-01](/img/enu2.PNG){: .align-left}

Get-NetUser

![source-01](/img/enu3.PNG){: .align-left}
