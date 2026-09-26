

# Link máy: [https://tryhackme.com/room/brooklynninenine](https://tryhackme.com/room/brooklynninenine)

## Mô tả: well nó là 1 máy khá là hiển nhiên dạy về Web Enumeration, Steganography và Privilege Escalation

## Véc tơ tấn công: Enumeration bằng Nmap → phát hiện cổng 80 và 22 → Steganography Credential hidden → credential → ssh → foothold → System Enumeration → SUID → root

## 1. Nmap Enumeration

```jsx
nmap -sC -sV <ip>
```


![[Pasted image 20260926230754.png]]
- Chạy xong bạn sẽ thấy 3 cổng mở
    - cổng 21 chạy FTP ⇒ có thể đăng nhập anonymous ⇒ (tôi sẽ nói cách Unintended sau)
    - cổng 22 chạy SSH ⇒ nó đang chạy ssh server. Hiện tại chưa có key hoặc username để brute force nên bỏ qua
    - cổng 80 chạy web server Apache ⇒ thứ duy nhất nên thử.

## 2. Web enumeration

![[Pasted image 20260926230850.png]]
- Vì không có hostname nên ta bỏ qua việc đi tìm vhost ⇒ bỏ vhost enumeration
- Directory Enumeration ⇒ đã chạy thử 1 số lần thì tôi thấy không tìm được gì thú vị
- Ctrl + U để xem mã nguồn thì tôi nhận thấy có dòng chữ steganography trong file html

![[Pasted image 20260926230922.png]]

⇒ Tìm hiểu trên mạng thì đây là kĩ thuật giấu trong file.

## 3. Steganography

- Tôi tải ảnh về bằng wget và hỏi AI thì nói gợi ý tôi dùng steghide hoặc stegseek với cái file này

![[Pasted image 20260926230952.png]]

⇒ CÓ CREDENTIALL!!!!!!

## 4. Privilege Escalation

- như 1 thói quen thì bạn có thể dùng python3 tạo server web tạm thời để đẩy tool enumeration như Linpeas các thứ
- Ở đây tôi phát hiện ra SUID lạ sau khi chạy sudo -l

![[Pasted image 20260926231006.png]]

⇒ Bạn có thể chạy nano ở quyền root ⇒ SUID này giúp mình lên được root

(Bằng chứng:)

![[Pasted image 20260926231026.png]]
- Về cơ bản các câu lệnh là

```jsx
sudo nano
Ctrl + R
Ctrl + X
rồi nhập: reset; sh 1>&0 2>&0

```

⇒ bạn có root shell

POC:

![[Pasted image 20260926231050.png]]

Bất ngờ chưa ⇒ có root.txt sudo rm -rf /* thôi =))

![[Pasted image 20260926231210.png]]

# Cách Unintended way

## Véc tơ tấn công: ftp anonymous login ⇒ get the username ⇒ brute force ssh ⇒ Có credential ⇒ foothold as jake ⇒ System Enumeration ⇒ SUID ⇒ root

## 1. FTP anonymous login

- Bạn có nhớ nmap cho bạn thấy FTP có thể đăng nhập anonymous không. Bạn có thể tìm được file đấy và nội dung như sau

![[Pasted image 20260926231229.png]]
- Nội dung file này cho thấy user tên jake có mật khẩu yếu

⇒ BRUTE FORCE!!!!!

## 2. Brute forcing jake ssh user account

```jsx
hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://<ip>
```


![[Pasted image 20260926231251.png]]

- welp nó nhanh hơn tôi tưởng tượng

## 3. Privilege Escalation

- Thử sudo -l lại lần nữa

![[Pasted image 20260926231317.png]]

- ta thấy less chạy với quyền root ⇒ SUID!!!! ⇒ lên trang gtfobins thì tôi thấy

![[Pasted image 20260926231338.png]]

lệnh lên root

⇒ we have the root!!!!

```jsx
sudo less /etc/hosts
sau đó ghi !/bin/bash => có root shell
```

POC

![[Pasted image 20260926231355.png]]