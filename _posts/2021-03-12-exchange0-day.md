---
layout: post
title:  "Mapping Exposed Exchange Servers in MV IP space"
date:   2021-03-12 14:07:20
categories: [Security Advisories ]
excerpt: "Last week, Microsoft released out-of-band updates to address multiple zero-day vulnerabilities which affect the on-premises version of the exchange server"

comments: true
---



Last week, Microsoft released out-of-band updates to address multiple zero-day vulnerabilities which affect the on-premises version of the exchange server. According to research firms, adversaries started mass exploitation of this vulnerability as early as Jan 2021. Cybercriminals are already exploiting this vulnerability to install ransomware.

##### Vulnerability detail

- **CVE-2021-26855:** Server-side request forgery vulnerability in Exchange which allowed the attacker to send arbitrary HTTP requests & authenticate as the Exchange server.
- **CVE-2021-26857:** Insecure deserialization vulnerability in the Unified Messaging service.
- **CVE-2021-27065:** Post-authentication arbitrary file write vulnerability in Exchange.
- **CVE-2021-26858:** Post-authentication arbitrary file write vulnerability in Exchange. Could use this vulnerability to write a file to any path on the server.

#### Mapping Exposed Exchange Servers in MV IP space:

Quick internet-wide scan of MV IP space shows few exchange servers exposed to the internet in the Maldives. Threat actors are performing mass-scans for the internet in search of vulnerable exchange servers. The number of exposed exchange servers in MV is very low compared to other parts of the world. If you are running exchange on-prem go patch now.


![source-01](/img/enu16111111.PNG){: .align-right}

Scan resulted in around 20 unique IPv4 addresses. 

![source-01](/img/screenshot167.PNG){: .align-left}

Half of the systems are coming from **AS7642** IP ranges

> IPs leased by companies/organizations from AS7642


##### Vulnerability detected by domain

![source-01](/img/screenshot1.PNG){: .align-left}

> Full Domain name redacted for privacy/security reasons


#### Detection / Investigation

Even if you already installed the updates and patched your exchange on-prem, it is highly recommended to investigate the server for any IOCs.

##### Scan exchange server for IOC's

- **CVE-2021-26855:** exploitation can be detected via the  Exchange HttpProxy logs
- **CVE-2021-26858:** exploitation can be detected via the Exchange log files
- **CVE-2021-26857:** exploitation can be detected via the Windows Application event logs
- **CVE-2021-27065:** exploitation can be detected via Exchange log files

Microsoft Exchange server team released a script for checking exchange server indicators of compromise. You can find the tool from here  
[IOC's detection tool](https://github.com/microsoft/CSS-Exchange/tree/main/Security)


#### Mitigations:

Microsoft has release out of band security updates for the following versions of exchange server
- Exchange Server 2013
- Exchange Server 2016
- Exchange Server 2019

##### Refference

- [IOC's detection tool](https://github.com/microsoft/CSS-Exchange/tree/main/Security)
- [Exchange Server Security Updates](https://techcommunity.microsoft.com/t5/exchange-team-blog/released-march-2021-exchange-server-security-updates/ba-p/2175901)



