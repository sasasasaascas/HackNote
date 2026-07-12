

# Các chức năng của Nmap

- [[Host Discovery]]: xem host nào đang hoạt động (rất phù hợp khi hệ thống cho 1 dải mạng)
- Port scanning: gõ cửa xem port nào mở
- Service scanning:  service nào?
- version scanning: version?
- OS scanning: OS?
- NSE scripts có khi phát hiện lỗ hổng như eternal blue của windows

# hạn chế của Nmap

- Vì nó là công cụ scan chủ động nên không thể tránh khỏi việc bị
	- IDS/IPS detections
	- Firewall đánh chặn
	- filtered output
- Công cụ này như cứt khi có thể không quét kĩ các cổng.


# Các câu lệnh

## scan cơ bản
```
nmap <ip>
```


## scan ports

### scan 1 số port 
```
nmap -p 21,22,3306,53,80,443,.... <ip>
```

### scan all port

```
nmap -p- <ip>
```


# đọc ở đây để xem tất cả các options của Nmap: [[Network Enumeration with Nmap - cheatsheet.pdf]]

