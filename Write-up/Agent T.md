

# Link box: [https://tryhackme.com/room/agentt](https://tryhackme.com/room/agentt)

## Véc tơ tấn công: Nmap → cổng 80 chạy php/8.1.0-dev → lỗ hổng RCE → POC → reverse shell → root → flag

### 1. Nmap

Kết quả Nmap

![[Pasted image 20260926231646.png]]

- Nhận xét về host:
    - cổng 80 đang chạy http web php 8.1.0-dev

### 2. Foothold as root

Header gợi ý:

![[Pasted image 20260926231719.png]]

- Nhận xét: đang chạy PHP/8.1.0-dev ⇒ lỗ hổng RCE ⇒ có thể lấy được shell và có được flag

Chúng ta có POC để áp dụng

[https://github.com/K3ysTr0K3R/PHP-8.1.0-dev-Backdoor](https://github.com/K3ysTr0K3R/PHP-8.1.0-dev-Backdoor)

- Tóm tắt cơ bản thì `User-Agentt backdoor → arbitrary code execution`
- Các bước thực hiện:

![[Pasted image 20260926231730.png]]

- Cho nó thực hiện revershell ⇒ ta có ROOOOOOOOOOOOOOOOOOT!!
- tìm được Flag ở /

![[Pasted image 20260926231741.png]]