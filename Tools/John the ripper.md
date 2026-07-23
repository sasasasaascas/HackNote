# John the Ripper

## Tổng quát

John the Ripper (JtR) là một bộ công cụ mã nguồn mở dùng để khôi phục hoặc bẻ khóa mật khẩu thông qua việc crack các giá trị hash và các tệp được bảo vệ bằng mật khẩu.

## Công dụng chính

- Crack hash
- Crack protected files
- Kiểm tra độ mạnh mật khẩu (Password Auditing)
- Hỗ trợ nhiều kiểu tấn công

---

# Cú pháp cơ bản

## Crack hash

```bash
john --wordlist=/path/to/wordlist --format=<hash-format> hash.txt
```

> **Lưu ý:** Có thể xác định hoặc dự đoán loại hash bằng các công cụ như `hashid`, `hash-identifier` hoặc `hashcat --example-hashes`.

Ví dụ:

```bash
hashid hash.txt
```

---

## Crack một số dịch vụ

### Crack file ZIP

```bash
zip2john file.zip > hash.txt

john --wordlist=/path/to/wordlist hash.txt
```

### Crack file RAR

```bash
rar2john file.rar > hash.txt

john --wordlist=/path/to/wordlist hash.txt
```

### Crack file PDF

```bash
pdf2john file.pdf > hash.txt

john --wordlist=/path/to/wordlist hash.txt
```

### Crack Microsoft Office

```bash
office2john file.docx > hash.txt

john --wordlist=/path/to/wordlist hash.txt
```

### Crack SSH Private Key

```bash
ssh2john id_rsa > hash.txt

john --wordlist=/path/to/wordlist hash.txt
```

### Crack KeePass Database

```bash
keepass2john database.kdbx > hash.txt

john --wordlist=/path/to/wordlist hash.txt
```

---


## Ngoài ra chúng ta còn có cả single mode

```bash
john --single hash.txt
```

