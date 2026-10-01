# Link máy: https://app.hackthebox.com/machines/Nibbles?sort_by=created_at&sort_type=desc

![[Pasted image 20260927170957.png]]
# véc tơ tấn công: Nmap -> cổng 80 http web apache -> xem source code -> có nibbleblog -> CVE-2015-6967 (hoặc chơi metasploit hoặc do it yourself) -> có foothold Nibbler -> sudo -l ra SUID root -> root


# 1. Nmap

![[Pasted image 20260926235156.png]]


- Nhận xét về host:
	- cổng 22 đang chạy SSH: chúng ta không quan tâm vì well chưa có gì để brute-force cả
	- cổng 80 đang chạy web-server apache: welp có lẽ đây là đường duy nhất rồi
# 2. Web Enumeration

![[Pasted image 20260926235457.png]]

- Vì không có DNS nên việc tìm subdomain không khả thi

- Chúng ta hãy thử Enumerate directory

```bash

feroxbuster --url http://<ip> --wordlist /path/to/list
```
- Ko có gì cả
![[Pasted image 20260927000857.png]]

- khi tôi Ctrl + U có gì đó vui

![[Pasted image 20260927000933.png]]


- hmm /nibbleblog
![[Pasted image 20260927001101.png]]

- thử enumerate directory tiếp
![[Pasted image 20260927001532.png]]


- nó chạy bản 4.0.3 Nibbleblog và có exploit 

https://github.com/advisories/GHSA-jc49-pxch-59wj

- Searchsploit cũng có script chạy trên metasploit:

![[Pasted image 20260927104857.png]]

# 3. Foothold as nibble


- Theo thông tin của Link trên, bạn có thể lấy được shell bằng cách tiêm php-pentest-monkey script vào plugins my_image của nibbleblog sau đó kích hoạt nó bạn sẽ được shell
- Dù vậy tôi có 2 phương pháp làm bài này
	- Cách 1: bạn do it yourself
	- Cách 2: sử dụng đồ có sẵn của metasploit
- Vì tôi bị lười nên tôi sẽ dùng metasploit


Dùng exploit
![[Pasted image 20260927105004.png]]


set options với default credentials 
![[Pasted image 20260927110545.png]]

=> đéo có tác dụng =)) điển hình

- Rồi giờ thì chơi tay không vậy


- Về lỗ hổng: chúng ta thấy Nibbleblog có phiên bản 4.0.3 dính 1 lỗ hổng là plugins có thể thay đổi được mã php => tiêm vào mã php reverse shell và có được shell.



Upload shell.php vào plugins my_image 
![[Pasted image 20260927164456.png]]


Đặt Listener
![[Pasted image 20260927164523.png]]



Giờ thì hãy trigger php thì: 
```
curl http://<ip>/nibbleblog/content/private/plugins/my_image/image.php
```

Vầ chúng ta có shell

![[Pasted image 20260927164819.png]]

Stable shell đó

```
python3 -c "import pty; pty.spawn('/bin/bash')"
Ctrl + Z 
stty raw -echo && fg

Enter 2 lần

export TERM=xterm
```

![[Pasted image 20260927165320.png]]


# 4. Privilege Escalation


```bash
sudo -l
```

![[Pasted image 20260927165839.png]]


- Chúng ta có thể chạy được file đường dẫn trên với quyền root mà không cần mật khẩu


![[Pasted image 20260927170123.png]]

=)) và mình chỉnh sửa được file đó =))

ta có các bước privilege escalation như sau:
- Bỏ file monitor.sh
- tạo monitor.sh mới bằng cách copy /bin/bash vào monitor.sh
- sử dụng sudo để chạy file này với quyền root

```
rm -rf monitor.sh
cp /bin/bash /home/nibbler/personal/stuff/monitor.sh
sudo /home/nibbler/personal/stuff/monitor.sh
```

=> chúng ta có được ROOOOT!!!!

![[Pasted image 20260927170555.png]]

# thế là xong giờ thì lấy flag và submit thôi.



# Các kỹ thuật sử dụng

- Enumeration Nmap
- Web Enumeration
- Reverse shell
- Local Privilege Escalation


|     |     |     |     |
| --- | --- | --- | --- |
|     |     |     |     |
|     |     |     |     |
|     |     |     |     |
