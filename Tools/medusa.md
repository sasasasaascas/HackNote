
## install:

```bash
sudo apt update
sudo apt install hydra
```

## Cú pháp cơ bản

```bash
medusa -h <IP/DNS> -u/-U <user/user.lst> -p/-P <pass/pass.lst> -M <module> -n <port>
```

- `-h` → target IP/DNS
- `-u` → 1 username
- `-U` → danh sách username
- `-p` → 1 password
- `-P` → danh sách password
- `-M` → module/service
- `-n` → port
- `-f` → dừng khi tìm được credential hợp lệ

---

## Tấn công SSH

```bash
medusa -h <IP> -u <user> -P <pass.lst> -M ssh
```

Chỉ định port:

```bash
medusa -h <IP> -u <user> -P <pass.lst> -M ssh -n <port>
```

Ví dụ:

```bash
medusa -h 10.10.10.10 -u admin -P passwords.txt -M ssh
```

---

## Tấn công FTP

```bash
medusa -h <IP> -u <user> -P <pass.lst> -M ftp
```

Chỉ định port:

```bash
medusa -h <IP> -u <user> -P <pass.lst> -M ftp -n <port>
```

Ví dụ:

```bash
medusa -h 10.10.10.10 -U users.txt -P passwords.txt -M ftp
```

---

## Tấn công RDP

```bash
medusa -h <IP> -u <user> -P <pass.lst> -M rdp
```

Chỉ định port:

```bash
medusa -h <IP> -u <user> -P <pass.lst> -M rdp -n 3389
```

 
 
 RDP module cần được hỗ trợ trong bản Medusa đang sử dụng


## Tấn công http-get
```bash
medusa -h www.example.com - U users.txt -P passwords.txt -M http -m GET
```



