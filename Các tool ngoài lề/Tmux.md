
# Tmux well nó là 1 cái terminal multiplexer đơn giản để bạn không cần phải làm ra cả đống cái window terminal lên.

# làm việc với session


## tạo session
```bash
tmux
tmux new
tmux new-session
tmux new -s sessionname
```

## gán session

```bash
tmux a
tmux att
tmux attach
tmux attach-session
tmux a -t sessionname
```

## xóa session
```bash
tmux kill-ses
tmux kill-session -t sessionname
```


# các phím tắt

## session

| Phím         | Chức năng         |
| ------------ | ----------------- |
| `Ctrl+b` `$` | Đổi tên session   |
| `Ctrl+b` `d` | Detach session    |
| `Ctrl+b` `)` | Session tiếp theo |
| `Ctrl+b` `(` | Session trước     |

## windows

|Phím|Chức năng|
|---|---|
|`Ctrl+b` `c`|Tạo window|
|`Ctrl+b` `n`|Window tiếp theo|
|`Ctrl+b` `p`|Window trước|
|`Ctrl+b` `l`|Window vừa dùng gần nhất|
|`Ctrl+b` `0-9`|Chuyển theo số|
|`Ctrl+b` `,`|Đổi tên window|
|`Ctrl+b` `.`|Đổi số window|
|`Ctrl+b` `f`|Tìm window|
|`Ctrl+b` `&`|Xóa window|
|`Ctrl+b` `w`|Danh sách window|
## Panes
### chia màn hình

| Phím         | Chức năng  |
| ------------ | ---------- |
| `Ctrl+b` `%` | Chia dọc   |
| `Ctrl+b` `"` | Chia ngang |


### Di chuyển
|Phím|Chức năng|
|---|---|
|`Ctrl+b` `←`|Sang pane trái|
|`Ctrl+b` `→`|Sang pane phải|
|`Ctrl+b` `↑`|Pane trên|
|`Ctrl+b` `↓`|Pane dưới|
|`Ctrl+b` `o`|Pane tiếp theo|
|`Ctrl+b` `;`|Quay lại pane trước|


### Quản lý
|Phím|Chức năng|
|---|---|
|`Ctrl+b` `{`|Di chuyển pane sang trái|
|`Ctrl+b` `}`|Di chuyển pane sang phải|
|`Ctrl+b` `!`|Tách pane thành window mới|
|`Ctrl+b` `x`|Đóng pane|
## Copy Mode

### vào copy mode:
```
Ctrl+b [
```

### Dán
```
Ctrl+b ]

```


### trong copy mode

| Phím    | Chức năng           |
| ------- | ------------------- |
| `Space` | Bắt đầu chọn        |
| `Enter` | Copy                |
| `Esc`   | Hủy                 |
| `g`     | Lên đầu             |
| `G`     | Xuống cuối          |
| `h`     | Trái                |
| `j`     | Xuống               |
| `k`     | Lên                 |
| `l`     | Phải                |
| `/`     | Tìm kiếm            |
| `#`     | Danh sách clipboard |
| `q`     | Thoát               |

### every day phím tắt
Ctrl+b c      Tạo window
Ctrl+b n      Window tiếp theo
Ctrl+b p      Window trước
Ctrl+b %      Chia dọc
Ctrl+b "      Chia ngang
Ctrl+b o      Sang pane tiếp theo
Ctrl+b x      Đóng pane
Ctrl+b d      Detach
tmux attach   Quay lại session
Ctrl+b [      Copy mode
Ctrl+b ]      Paste


==xin chào khiêm== 


![[Getting Started - cheatsheet.pdf]]