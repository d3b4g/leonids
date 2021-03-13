Last week, Microsoft release out of band updates to address multiple zero day vulnerabilities which affect on-premises version of exchange server. According to research firms adversaries started mass exploitation of this vulnerability as early as Jan 2021. Most of these are APT groups interested in cyber espionage, according to the latest reports, cybercriminals are exploiting this vulnerability to  install ransomewared dubbed "DearCry"

Vulnerability detail

- **CVE-2021-26855:** Server-side request forgery (SSRF) vulnerability in Exchange which allowed the attacker to send arbitrary HTTP requests & authenticate as the Exchange server.
- **CVE-2021-26857:** Insecure deserialization vulnerability in the Unified Messaging service.
- **CVE-2021-27065:** Post-authentication arbitrary file write vulnerability in Exchange.
- **CVE-2021-26858:** Post-authentication arbitrary file write vulnerability in Exchange. Could use this vulnerability to write a file to any path on the server.

#### Exchange Servers in MV IP space:

Quick internet wide scan of MV IP space shows few exchange servers exposed to internet in Maldives. 


![source-01](/img/enu16111111.PNG){: .align-left}

Vulnerable to latest Exchange 0day exploits
![source-01](/img/enu161111112.PNG){: .align-left}


![source-01](/img/screenshot1.PNG){: .align-left}

#### Detection / Investigation

Even if you already installed the updates and patched your exchange on prem, it is highly encouraged to investigate the server for any IOC's.

##### Scan exchange server for IOC's

- **CVE-2021-26855:** exploitation can be detected via the  Exchange HttpProxy logs
- **CVE-2021-26858:** exploitation can be detected via the Exchange log files
- **CVE-2021-26857:** exploitation can be detected via the Windows Application event logs
- **CVE-2021-27065:** exploitation can be detected via Exchange log files


Microsoft Exchange server team released a script for checking HAFNIUM indicators of compromise. You can find the tool from here  
[IOC's detection tool](https://github.com/microsoft/CSS-Exchange/tree/main/Security)


#### Mitigations:
Microsoft has release out of band security updates for the following versions of exchange server

- Exchange Server 2013
- Exchange Server 2016
- Exchange Server 2019

##### Refference

[IOC's detection tool](https://github.com/microsoft/CSS-Exchange/tree/main/Security)
[Exchange Server Security Updates](https://techcommunity.microsoft.com/t5/exchange-team-blog/released-march-2021-exchange-server-security-updates/ba-p/2175901)



