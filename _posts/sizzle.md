

#### Background


**Sizzler** is a a really tough windows box. 

#### Enumeration

To begin the enumeration process, a **TCP/UDP** port scan was run against the target using nmap. The purpose of scan is to quickly determine which ports are open and which services are running. 

###### OS Information
Based on the system we know it is running some version of windows server

###### Open Ports:
In addition to usual ports, 22 and 80 UDP port 161 is open. UDP 161 is the standard SNMP port.There is also a filtered ftp port showing.
```python
  sizzle nmap -sC -sV 10.10.10.103 -vvv
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 11:18 EDT
PORT     STATE SERVICE       REASON          VERSION
21/tcp   open  ftp           syn-ack ttl 127 Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)

80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
443/tcp  open  ssl/http      syn-ack ttl 127 Microsoft IIS httpd 10.0
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)

```
#### Notes:
- There are a lot of ports open and by looking at the open ports we know it is most likely a windows active directory setup.

#### HTTP - TCP 80
Gobuster did find an interesting directory **certenroll**  which is microsoft certificate authority enrolement service.
```python

/aspnet_client (Status: 301)
/certenroll (Status: 301)
/Images (Status: 301)
/images (Status: 301)
/index.html (Status: 200)
```
But visiting that directory show us 403 forbidden

![source-01](/img/sizzle1.PNG){: .align-left}



#### SMB - TCP 445
Using SMB client i was able to get bunch of shares.

```python

sizzle smbclient -N -L \\\\sizzle.htb.local

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        CertEnroll      Disk      Active Directory Certificate Services share
        Department Shares Disk
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share
        Operations      Disk
        SYSVOL          Disk      Logon server share
SMB1 disabled -- no workgroup available
```
