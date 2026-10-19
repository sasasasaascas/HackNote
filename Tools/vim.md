

# Vim Cheat Sheet

> Vim hoạt động theo **mode**. Đây là điểm quan trọng nhất cần hiểu trước khi sử dụng.
> 
> Nếu không biết mình đang ở mode nào → nhấn `Esc` để quay về **Normal Mode**.

---

# 1. Các Mode trong Vim

## Normal Mode

Mode mặc định, dùng để:

- Di chuyển
    
- Xóa
    
- Copy/Paste
    
- Tìm kiếm
    
- Chạy command
    

Quay về Normal Mode:

```
Esc
```

---

## Insert Mode

Dùng để nhập/chỉnh sửa text.

```
i       Insert trước con trỏ
a       Insert sau con trỏ
I       Insert đầu dòng
A       Insert cuối dòng

o       Tạo dòng mới bên dưới
O       Tạo dòng mới bên trên
```

Thoát Insert Mode:

```
Esc
```

---

## Visual Mode

Dùng để chọn text.

```
v       Chọn theo ký tự
V       Chọn cả dòng
Ctrl+v  Chọn theo block/cột
```

Sau khi chọn có thể:

```
y       Copy
d       Delete/Cut
```

---

# 2. Mở File

```
vim file.txt
```

Ví dụ:

```
vim /etc/hosts
vim exploit.py
vim config.php
```

Mở tại một dòng cụ thể:

```
vim +50 file.txt
```

→ Mở tại dòng `50`.

---

# 3. Lưu và Thoát

Nhấn:

```
Esc
```

Sau đó:

```
:w          Save
:q          Quit
:wq         Save + Quit
:x          Save + Quit

:q!         Quit không lưu
:wq!        Force Save + Quit
```

Cheat:

```
:w      write
:q      quit
:wq     write + quit
:q!     fuck this shit
```

---

# 4. Di chuyển

## Cơ bản

```
h       ←
j       ↓
k       ↑
l       →
```

Hoặc dùng arrow keys:

```
← ↑ ↓ →
```

---

## Theo từ

```
w       Sang đầu từ tiếp theo
b       Về đầu từ trước
e       Sang cuối từ
```

Ví dụ:

```
3w
```

→ Đi sang 3 từ.

---

## Trong dòng

```
0       Đầu dòng
^       Ký tự đầu tiên của dòng
$       Cuối dòng
```

---

## Trong file

```
gg      Đầu file
G       Cuối file
```

Đi tới dòng cụ thể:

```
50G
```

→ Đi tới dòng `50`.

Hoặc:

```
:50
```

---

# 5. Page Navigation

```
Ctrl+f      Xuống 1 page
Ctrl+b      Lên 1 page

Ctrl+d      Xuống nửa page
Ctrl+u      Lên nửa page
```

> Trong Vim, `Ctrl+b` là page up.
> 
> Nếu đang chạy Vim bên trong tmux thì `Ctrl+b` cũng là prefix của tmux. Tmux chỉ bắt prefix khi bạn thực hiện tổ hợp prefix tương ứng.

---

# 6. Xóa Text

## Xóa ký tự

```
x       Xóa ký tự dưới con trỏ
X       Xóa ký tự trước con trỏ
```

---

## Xóa dòng

```
dd      Xóa cả dòng
```

Nhiều dòng:

```
5dd
```

→ Xóa 5 dòng.

---

## Xóa theo từ

```
dw      Xóa từ vị trí hiện tại
```

```
d$
```

→ Xóa từ con trỏ tới cuối dòng.

```
d0
```

→ Xóa từ con trỏ tới đầu dòng.

---

# 7. Copy / Paste

Trong Vim:

```
y = yank = copy
p = paste
```

Copy dòng:

```
yy
```

Paste xuống dưới:

```
p
```

Paste lên trên:

```
P
```

Copy nhiều dòng:

```
5yy
```

→ Copy 5 dòng.

---

## Cut

Trong Vim, thao tác delete cũng đưa nội dung vào register.

```
dd
```

Sau đó:

```
p
```

→ Gần giống **cut + paste**.

Ví dụ di chuyển một dòng:

```
dd
```

Di chuyển tới vị trí mới:

```
p
```

---

# 8. Undo / Redo

```
u           Undo

Ctrl+r      Redo
```

---

# 9. Search

Tìm từ phía dưới:

```
/password
```

Enter.

Sau đó:

```
n       Kết quả tiếp theo
N       Kết quả trước
```

Tìm ngược lên:

```
?password
```

---

## Tìm từ đang nằm dưới con trỏ

Đặt con trỏ lên từ:

```
*
```

→ Tìm occurrence tiếp theo.

```
#
```

→ Tìm ngược lại.

---

# 10. Search & Replace

Thay lần đầu tiên trong dòng hiện tại:

```
:s/old/new/
```

Thay tất cả trong dòng:

```
:s/old/new/g
```

Thay toàn bộ file:

```
:%s/old/new/g
```

Ví dụ:

```
:%s/localhost/10.10.11.20/g
```

---

## Hỏi trước khi Replace

```
:%s/old/new/gc
```

Vim sẽ hỏi từng occurrence.

```
y       yes
n       no
a       all
q       quit
```

---

# 11. Visual Selection

## Chọn text

```
v
```

Di chuyển bằng:

```
h j k l
```

Sau đó:

```
y       Copy
d       Delete
```

---

## Chọn cả dòng

```
V
```

Sau đó di chuyển:

```
j
j
j
```

Rồi:

```
y
```

---

## Block Selection

```
Ctrl+v
```

Hữu ích khi muốn chọn nhiều cột cùng lúc.

---

# 12. Edit nhanh

Thay một ký tự:

```
r
```

Ví dụ con trỏ đang ở:

```
hellp
    ^
```

Nhấn:

```
ro
```

→

```
hello
```

---

Thay text liên tục:

```
R
```

→ Replace Mode.

---

Thay cả từ:

```
cw
```

Nhập từ mới rồi:

```
Esc
```

---

Thay toàn bộ dòng:

```
cc
```

---

# 13. Number + Command

Vim cho phép:

```
<number><command>
```

Ví dụ:

```
5j
```

→ Xuống 5 dòng.

```
3w
```

→ Sang 3 từ.

```
10dd
```

→ Xóa 10 dòng.

```
5yy
```

→ Copy 5 dòng.

```
20G
```

→ Đi tới dòng 20.

---

# 14. Line Number

Hiện số dòng:

```
:set number
```

Viết ngắn:

```
:set nu
```

Tắt:

```
:set nonumber
```

Hoặc:

```
:set nonu
```

---

# 15. Paste Code không bị Indent loạn

Một vấn đề khá hay gặp khi paste exploit/config/script vào Vim.

Bật paste mode:

```
:set paste
```

Sau đó:

```
i
```

Paste code.

Xong:

```
Esc
:set nopaste
```

---

# 16. Chạy Command Linux từ Vim

Không cần thoát Vim.

```
:!command
```

Ví dụ:

```
:!ls
```

```
:!whoami
```

```
:!python3 exploit.py
```

```
:!ip addr
```

Nhấn Enter để quay lại Vim.

---

# 17. Mở Shell từ Vim

```
:shell
```

Bây giờ đang ở shell.

Muốn quay lại Vim:

```
exit
```

---

# 18. Đọc File khác vào File hiện tại

```
:r file
```

Ví dụ:

```
:r payload.txt
```

→ Nội dung `payload.txt` được chèn vào file hiện tại.

Có thể đọc output của command:

```
:r !command
```

Ví dụ:

```
:r !whoami
```

---

# 19. Cheat Sheet

## Mode

```
Esc         Normal Mode

i           Insert
a           Append
o           New line below
O           New line above

v           Visual
V           Visual Line
Ctrl+v      Visual Block
```

## Save / Exit

```
:w          Save
:q          Quit
:wq         Save + Quit
:q!         Quit không save
```

## Navigation

```
h j k l     ← ↓ ↑ →

w           Next word
b           Previous word

0           Đầu dòng
$           Cuối dòng

gg          Đầu file
G           Cuối file

50G         Dòng 50
```

## Edit

```
x           Delete character

dd          Delete line
5dd         Delete 5 lines

yy          Copy line
5yy         Copy 5 lines

p           Paste

u           Undo
Ctrl+r      Redo
```

## Search

```
/text       Search
n           Next
N           Previous

*           Search word under cursor
```

## Replace

```
:s/a/b/         Replace trong dòng
:s/a/b/g        Replace tất cả trong dòng
:%s/a/b/g       Replace toàn file
:%s/a/b/gc      Replace + confirm
```

---

# Những lệnh quan trọng nhất

Nếu chỉ dùng Vim khi SSH vào box và không có `nano`, chỉ cần nhớ:

```
i           Bắt đầu nhập
Esc         Ngừng nhập / về Normal Mode

:w          Save
:q          Quit
:wq         Save + Quit
:q!         Quit không save

h j k l     Di chuyển
gg          Đầu file
G           Cuối file

dd          Xóa dòng
yy          Copy dòng
p           Paste

u           Undo
Ctrl+r      Redo

/text       Search
n           Next result

:%s/a/b/g   Replace toàn file
```

---

# Vim Survival Mode

Khi vào một máy lạ và **bắt buộc phải dùng Vim**, nhớ đúng chuỗi này:

```
vim file
    ↓
i
    ↓
Edit...
    ↓
Esc
    ↓
:wq
```

Nếu sửa nát file và muốn chạy:

```
Esc
:q!
```

> **Quy tắc cứu mạng:** Không biết Vim đang làm cái gì → `Esc`.