
# Enumeration with Nmap


Nmap
![[Pasted image 20261015200032.png]]

- Nhận xét host:
	- cổng 22 SSH không thể làm được gì
	- cổng 80 HTTP đang chạy server apache (Debian)


Lưu host tìm được
![[Pasted image 20261015204822.png]]
# Web Enumeration


sử dụng ffuf để enumerate vhost thì tìm được *research.bedside.htb*

![[Pasted image 20261015204755.png]]


Lưu vhost
![[Pasted image 20261015204915.png]]


hmm khi gửi file thì header lại là pdfminer.six
![[Pasted image 20261015205123.png]]



lỗ hổng LFI trên cổng 3000


# shell as datawrangler




# shell as developer
![[Pasted image 20261015214859.png]]


