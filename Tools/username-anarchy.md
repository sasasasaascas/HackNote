## Bản chất

Tool dùng để **tạo ra nhiều biến thể username từ tên của một người**.

Ví dụ:

```
John Smith
↓
john
smith
johnsmith
john.smith
j.smith
jsmith
smithj
...
```

Thường dùng trong **username enumeration / password spraying / brute-force**, khi biết tên thật của mục tiêu nhưng chưa biết username.

---

## Cài đặt

Trên Parrot/Debian:

```bash
sudo apt update
sudo apt install username-anarchy
```

Nếu repo không có:

```bash
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy
```

Kiểm tra:

```bash
./username-anarchy --help
```

---

## Cú pháp cơ bản

```bash
username-anarchy <name>
```

Ví dụ:

```bash
username-anarchy "John Smith"
```

→ tạo ra danh sách các username có thể có.

---

## Từ file chứa tên

```bash
username-anarchy -i names.txt
```

Ví dụ:

```
John Smith
Alice Johnson
Bob Williams
```

→ sinh username cho từng người.

---

## Lưu kết quả

```
username-anarchy "John Smith" > users.txt
```