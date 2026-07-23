# Hashcat

Hashcat là một trong những công cụ crack mật khẩu phổ biến và mạnh mẽ nhất hiện nay. Khác với [[John the Ripper]], Hashcat được tối ưu để tận dụng **GPU** (CUDA, OpenCL, HIP...) nhằm tăng tốc quá trình bẻ khóa mật khẩu, giúp tốc độ crack nhanh hơn rất nhiều so với chỉ sử dụng CPU.

## Đặc điểm

- Hỗ trợ tăng tốc bằng **GPU**, đồng thời vẫn có thể chạy trên CPU.
- Hỗ trợ hàng trăm thuật toán hash khác nhau.
- Có nhiều chế độ tấn công như:
  - Dictionary Attack
  - Brute Force (Mask Attack)
  - Rule-based Attack
  - Combination Attack
  - Hybrid Attack
- Hỗ trợ **salted hash** và **unsalted hash**.
- Có thể khôi phục (resume) phiên làm việc nếu quá trình crack bị gián đoạn.

## So sánh với John the Ripper

| Hashcat | John the Ripper |
|---------|------------------|
| Tối ưu cho GPU | Chủ yếu tối ưu cho CPU (có hỗ trợ GPU ở Jumbo) |
| Tốc độ rất cao khi có GPU mạnh | Thường chậm hơn trong các bài toán GPU |
| Hỗ trợ rất nhiều loại hash | Hỗ trợ rất nhiều loại hash |
| Mạnh về Mask Attack và Rule-based Attack | Mạnh về Single Crack Mode và khả năng tự sinh mật khẩu từ thông tin người dùng |
| Phù hợp cho các tác vụ cần hiệu năng cao | Phù hợp cho pentest và CTF nhờ dễ sử dụng |

> **Lưu ý:** Hashcat không "hỗ trợ nhiều loại hash hơn" John the Ripper một cách tuyệt đối. Cả hai đều hỗ trợ hàng trăm loại hash và thường có mức độ bao phủ tương đương. Điểm mạnh lớn nhất của Hashcat là khả năng tận dụng GPU để tăng tốc và các chế độ tấn công linh hoạt, trong khi John the Ripper nổi bật với Single Crack Mode và khả năng tự động khai thác thông tin người dùng.



# Cách dùng cơ bản

```bash
hashcat -a <mode> -m <hashid>  /path/to/hash /path/to/wordlist
```


## Các Attack Mode

Hashcat hỗ trợ nhiều chế độ tấn công khác nhau. Trong thực tế, bạn sẽ sử dụng chủ yếu 5 mode dưới đây.

| Mode | Tên                    | Mô tả                               |
| ---- | ---------------------- | ----------------------------------- |
| `0`  | Straight (Dictionary)  | Thử từng mật khẩu trong wordlist.   |
| `1`  | Combination            | Ghép hai wordlist lại với nhau.     |
| `3`  | Brute-force (Mask)     | Sinh mật khẩu theo mẫu (mask).      |
| `6`  | Hybrid Wordlist + Mask | Wordlist trước, thêm mask phía sau. |
| `7`  | Hybrid Mask + Wordlist | Mask trước, wordlist phía sau.      |

### Mode 0 - Dictionary Attack

Đây là mode được sử dụng nhiều nhất.

```bash
hashcat -a 0 -m 1000 hash.txt rockyou.txt
```

Ví dụ Hashcat sẽ thử:

```
password
admin
123456
qwerty
...
```

---

### Mode 1 - Combination Attack

Ghép từng từ trong hai wordlist.

```bash
hashcat -a 1 -m 1000 hash.txt list1.txt list2.txt
```

Ví dụ:

`list1.txt`

```
admin
root
```

`list2.txt`

```
123
2024
```

Hashcat sẽ thử:

```
admin123
admin2024
root123
root2024
```

---

### Mode 3 - Brute-force (Mask Attack)

Sinh mật khẩu theo một mẫu xác định.

```bash
hashcat -a 3 -m 1000 hash.txt ?l?l?l?l?d?d
```

Ví dụ:

```
abcd12
test99
hack01
```

Các ký hiệu thường dùng:

| Ký hiệu | Ý nghĩa |
|---------|----------|
| `?l` | Chữ thường (`a-z`) |
| `?u` | Chữ hoa (`A-Z`) |
| `?d` | Số (`0-9`) |
| `?s` | Ký tự đặc biệt |
| `?a` | Tất cả các ký tự trên |

---

### Mode 6 - Hybrid (Wordlist + Mask)

Thêm mask vào cuối mỗi từ trong wordlist.

```bash
hashcat -a 6 -m 1000 hash.txt rockyou.txt ?d?d
```

Ví dụ:

```
password01
password99
admin12
```

---

### Mode 7 - Hybrid (Mask + Wordlist)

Thêm mask vào đầu mỗi từ trong wordlist.

```bash
hashcat -a 7 -m 1000 hash.txt ?d?d rockyou.txt
```

Ví dụ:

```
01password
99password
12admin
```

## Tóm tắt

| Attack Mode | Khi nào dùng |
|--------------|------------------------------|
| `0` | Có wordlist. |
| `1` | Muốn ghép hai wordlist. |
| `3` | Biết cấu trúc mật khẩu. |
| `6` | Biết mật khẩu có hậu tố (ví dụ: `Password123`). |
| `7` | Biết mật khẩu có tiền tố (ví dụ: `2024Password`). |