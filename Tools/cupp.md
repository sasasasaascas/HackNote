## Bản chất

**CUpp (Common User Passwords Profiler)** dùng để **tạo password wordlist theo thông tin về một người**.

Ví dụ biết:

```
Name: John
Nickname: Johnny
Birthday: 1998
Pet: Rex
Partner: Alice
```

→ CUpp có thể tạo ra các biến thể password dựa trên những thông tin đó.

---

## Cài đặt

Trên Parrot/Debian:

```
sudo apt update
sudo apt install cupp
```

Kiểm tra:

```
cupp -h
```

Nếu repo không có:

```
git clone https://github.com/Mebus/cupp.git
cd cupp
python3 cupp.py -h
```

---

## Cú pháp cơ bản

```
cupp -i
```

`-i` → tạo wordlist **interactively**, nhập thông tin về target.

Ví dụ:

```
First Name: John
Surname: Smith
Nickname: Johnny
Birthdate: 1998
...
```

Sau đó CUpp tạo ra password list.

---

## Các option thường dùng

```
cupp -i
```

Interactive mode – tạo wordlist từ thông tin target.

```
cupp -w
```

Chế độ tạo wordlist từ một wordlist có sẵn.

```
cupp -l
```

Liệt kê các file wordlist được hỗ trợ.

```
cupp -a
```

Tải các wordlist từ repository của CUpp.

```
cupp -h
```

Hiển thị help.



- về cơ bản thì cupp để tạo ra wordlist liên quan đến mục tiêu giúp thuận tiện trong việc brute-force mật khẩu


