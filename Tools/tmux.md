# Tmux Cheat Sheet

> **Prefix mặc định:** `Ctrl + b`
> 
> Khi thấy `Ctrl+b → c`: nhấn `Ctrl+b`, **thả ra**, sau đó nhấn `c`.

---

## 1. Session

Session là một phiên làm việc của tmux. Khi **detach**, các chương trình bên trong session vẫn tiếp tục chạy.

```
# Tạo session mới
tmux

# Tạo session có tên
tmux new -s htb

# Xem danh sách session
tmux ls

# Attach vào session
tmux attach -t htb

# Cách viết ngắn
tmux a -t htb

# Xóa session
tmux kill-session -t htb

# Xóa toàn bộ tmux server/session
tmux kill-server
```

### Detach khỏi session

```
Ctrl+b → d
```

> Detach **không kill session**. Các chương trình như SSH, listener, web server,... vẫn tiếp tục chạy.

---

## 2. Window

Window có thể hiểu tương tự **tab** trong terminal.

|Phím|Chức năng|
|---|---|
|`Ctrl+b → c`|Tạo window mới|
|`Ctrl+b → n`|Window tiếp theo|
|`Ctrl+b → p`|Window trước|
|`Ctrl+b → 0..9`|Chuyển tới window theo số|
|`Ctrl+b → w`|Hiện danh sách window|
|`Ctrl+b → ,`|Đổi tên window|
|`Ctrl+b → &`|Kill window hiện tại|

### Ví dụ workflow HTB

```
0:nmap
1:web
2:shell
3:privesc
```

Để rename window:

```
Ctrl+b → ,
```

Sau đó nhập tên:

```
nmap
```

---

## 3. Pane

Pane dùng để **chia một window thành nhiều terminal nhỏ**.

### Chia màn hình

```
Ctrl+b → %      Chia trái/phải
Ctrl+b → "      Chia trên/dưới
```

Ví dụ:

```
┌───────────────────────┬───────────────────────┐
│                       │                       │
│     Reverse Shell     │      Enumeration      │
│                       │                       │
├───────────────────────┤                       │
│                       │                       │
│       Listener        │                       │
│                       │                       │
└───────────────────────┴───────────────────────┘
```

### Di chuyển giữa các pane

```
Ctrl+b → ←
Ctrl+b → →
Ctrl+b → ↑
Ctrl+b → ↓
```

### Zoom pane

```
Ctrl+b → z
```

Pane hiện tại sẽ chiếm toàn bộ window.

Nhấn lại:

```
Ctrl+b → z
```

để quay lại layout cũ.

### Đóng pane

```
Ctrl+b → x
```

Hoặc đơn giản:

```
exit
```

---

## 4. Resize Pane

Vào command mode:

```
Ctrl+b → :
```

Sau đó:

```
resize-pane -L 5
resize-pane -R 5
resize-pane -U 5
resize-pane -D 5
```

Trong đó:

```
-L    Left
-R    Right
-U    Up
-D    Down
```

Ví dụ:

```
resize-pane -R 10
```

→ Tăng kích thước pane sang phải 10 ô.

---

## 5. Scroll / Copy Mode

Để xem lại output dài:

```
Ctrl+b → [
```

Sau đó sử dụng:

```
↑ ↓
PageUp
PageDown
```

Thoát copy mode:

```
q
```

---

## 6. Command Mode

Mở command mode:

```
Ctrl+b → :
```

Một số command:

```
:new-window
:split-window
:kill-pane
:kill-window
:rename-window
```

---

## 7. Session Workflow

### Tạo workspace HTB

```
tmux new -s htb
```

Có thể chia thành:

```
Session: htb

├── Window 0: nmap
│
├── Window 1: web
│   ├── Pane 1: ffuf / feroxbuster
│   └── Pane 2: curl / testing
│
├── Window 2: shell
│   ├── Pane 1: nc listener
│   └── Pane 2: reverse shell
│
└── Window 3: privesc
```

Detach:

```
Ctrl+b → d
```

Sau này quay lại:

```
tmux a -t htb
```

---

## 8. Cheat Sheet

### Session

```
tmux new -s htb        Tạo session
tmux ls                List session
tmux a -t htb          Attach
tmux kill-session -t   Kill session

Ctrl+b → d             Detach
```

### Window

```
Ctrl+b → c             New window
Ctrl+b → n             Next window
Ctrl+b → p             Previous window
Ctrl+b → 0..9          Chọn window
Ctrl+b → w             List windows
Ctrl+b → ,             Rename window
Ctrl+b → &             Kill window
```

### Pane

```
Ctrl+b → %             Split trái/phải
Ctrl+b → "             Split trên/dưới

Ctrl+b → ← ↑ ↓ →       Di chuyển pane

Ctrl+b → z             Zoom/unzoom pane
Ctrl+b → x             Kill pane
```

### Scroll

```
Ctrl+b → [             Copy/scroll mode
↑ ↓                    Scroll
PageUp / PageDown      Scroll nhanh
q                      Thoát
```

### Command

```
Ctrl+b → :             Command mode
```

---

# Những phím quan trọng nhất

Nếu không muốn nhớ quá nhiều thì chỉ cần thuộc:

```
Ctrl+b → c       New window

Ctrl+b → %       Split trái/phải
Ctrl+b → "       Split trên/dưới
Ctrl+b → arrows  Chuyển pane
Ctrl+b → z       Zoom pane

Ctrl+b → [       Scroll

Ctrl+b → d       Detach
Ctrl+b → w       Xem windows
```

> **Nhớ một câu:** `Ctrl+b` là prefix — gần như mọi thao tác tmux đều bắt đầu từ đây.