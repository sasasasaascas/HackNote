






```nmap
NSE Timing: About 99.94% done; ETC: 01:10 (0:00:00 remaining)  
Nmap scan report for 10.129.231.149  
Host is up (0.047s latency).  
Not shown: 988 filtered tcp ports (no-response)  
PORT     STATE SERVICE       VERSION  
53/tcp   open  domain        Simple DNS Plus  
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-13 01:08:04Z)  
135/tcp  open  msrpc         Microsoft Windows RPC  
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn  
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)  
|_ssl-date: 2026-11-13T01:09:25+00:00; +6h58m33s from scanner time.  
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb  
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb  
| Not valid before: 2024-08-22T20:24:16  
|_Not valid after:  2025-08-22T20:24:16  
445/tcp  open  microsoft-ds?  
464/tcp  open  kpasswd5?  
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0  
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)  
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb  
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb  
| Not valid before: 2024-08-22T20:24:16  
|_Not valid after:  2025-08-22T20:24:16  
|_ssl-date: 2026-11-13T01:09:25+00:00; +6h58m33s from scanner time.  
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)  
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb  
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb  
| Not valid before: 2024-08-22T20:24:16  
|_Not valid after:  2025-08-22T20:24:16  
|_ssl-date: 2026-11-13T01:09:25+00:00; +6h58m33s from scanner time.  
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)  
|_ssl-date: 2026-11-13T01:09:25+00:00; +6h58m33s from scanner time.  
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb  
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb  
| Not valid before: 2024-08-22T20:24:16  
|_Not valid after:  2025-08-22T20:24:16  
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)  
|_http-title: Not Found  
|_http-server-header: Microsoft-HTTPAPI/2.0  
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows  
  
Host script results:  
| smb2-security-mode:  
|   3:1:1:  
|_    Message signing enabled and required  
|_clock-skew: mean: 6h58m33s, deviation: 0s, median: 6h58m32s  
| smb2-time:  
|   date: 2026-11-13T01:08:45  
|_  start_date: N/A
```