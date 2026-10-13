
# Lời nói đầu: đây thực sự là bài đầu tiên tôi giải sau khi học được Hackthebox 1 lúc đấy





# Véc tơ tấn công: Nmap -> web-enumearation -> PCAP file credential stealing -> nathans credential -> FTP -> reuse credential -> SSH -> shell as Nathan -> cap binary exploitation python3.8 -> setuid(0) -> shell as root


# Enumeration


![[Pasted image 20261013144105.png]]


Nhận xét:

- Cổng 21 đang mở FTP -> thông thường tấn công bằng brute-force
- Cổng 22 đang mở SSH -> thông thường thì sẽ tấn công bằng brute-force
- Cổng 80 đang chạy web server gunicorn -> thứ đáng quan tâm

# Web Enumeration

- Theo những gì tôi thu thập được thì không có 1 cái DNS nào cho subdomain-enumeration nên là tôi sẽ skip việc đó


Giao diện web
![[Pasted image 20261013144458.png]]

- Chúng ta đang ở 1 session của Nathan

direction /data/1
![[Pasted image 20261013144614.png]]



Khi thử thay đổi directory tôi đã phát hiện rằng trang web này bị 1 lỗ hổng IDOR khiến cho mình có thể đọc được cái file PCAP khác mà không cần permissions

/data/0
![[Pasted image 20261013144755.png]]




khi tôi nhấn download -> nhận được file pcap chứa 1 số thông tin quan trọng


![[Pasted image 20261013144933.png]]



![[Pasted image 20261013145033.png]]



Bằng cách phân tích các packet -> tôi có được Nathan credentials vào trong service FTP

nathan:Buck3tH4TF0RM3!
![[Pasted image 20261013145327.png]]

tìm được user.txt flag



# shell as nathan


- Trong lúc đang bí thì tôi thử sử dụng lại credential trên với ssh xem sao và nó đã vào được

![[Pasted image 20261013145519.png]]

chúng ta đã có shell as nathan

# shell as root

chạy linpeas
![[Pasted image 20261013145941.png]]


![[Pasted image 20261013150147.png]]


Khi chạy Linpeas tôi thấy /usr/bin/python3.8 đang có special capabilities nên có thể abuse nó và lên được root cụ thể bạn có thể sử dụng python để đưa uid về 0 (lên root)

![[Pasted image 20261013150414.png]]



![[Pasted image 20261013150944.png]]
=> bạn có quyền root rồi => lấy được flag



# Cách khắc phục


- Tắt capabilities không cần thiết
- Khắc phục IDOR
- sử dụng SFTP thay vì FTP để không bị sniff packets
- Prevent reusing credentials.