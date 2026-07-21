
# Công dụng của hydra

- Nó là công cụ để "**Thử tất cả các credentials**"
- Password Spraying (thử các user xem 1 mật khẩu đó là của user nào)
# Hướng dẫn sử dụng

![[Login Brute Forcing - cheatsheet.pdf]]


# Các câu lệnh cơ bản

```
Lưu ý các flag

-s <port> # để  cho biết port đang tấn công
-t <number> số  threads tối đa bạn có thể có
-l <tên nhất định>
-L <danh sách tên>
-P <danh sách mật khẩu>
-p <mật khẩu>
```


# tổng quát
```
hydra -l/-L username/file-username -p/-P password/file-password service://server-ip -s port
```



## tấn công ssh

```bash
hydra -l admin -P password.txt ssh://192.111.222.33
```

## tấn công ftp

```bash
hydra -L usr_name.txt -P password.txt ftp://11.22.33.44 -s 2222
```


## tấn công imap

```bash
hydra -L usr_name.txt -P password.txt imap://11.22.33.44 -s 2222
```

## tấn công http-post-form

```bash
hydra -l admin -P /path/to/password_list.txt 127.0.0.1 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"
```


# Lưu ý: Nhớ phải đúng format, 1 số dịch vụ có thể config để chống thử nhiều lần hoặc trả về fake response.
