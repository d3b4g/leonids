Last week, Microsoft release out of band updates to address multiple zero day vulnerabilities which affect on-premises version of exchange server. According to research firms adversaries started mass exploitation of this vulnerability as early as Jan 2021.

Vulnerability detail

CVE-2021-26855
CVE-2021-26857
CVE-2021-27065
CVE-2021-26858

Exchange Servers in MV IP space:



Mitigations:

If you are running Exchange server on prem 
1- Deploy updates
2- Invetigate for IOC's: analyze exchange server logs for indicators of compromise. Microsoft released bunch of IOC's to help defenders. https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/Sample%20Data/Feeds/MSTICIoCs-ExchangeServerVulnerabilitiesDisclosedMarch2021.csv
3- If you are unable to immediately apply updates, follow Microsoft’s alternative mitigations
