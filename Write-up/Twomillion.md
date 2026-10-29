

![[Pasted image 20261016231159.png]]


# Véc tơ tấn công

www-data: Nmap -> Web Enumeration + source code analyst -> API abuse + IDOR -> code injection -> reverse shell
admin: env enumeration -> password -> admin creds

root:  -> Overlays FS kernel level exploit CVE-2023-0386


# Nmap 

![[Pasted image 20261016231330.png]]


- Nhận xét về host:
	- cổng 22: chưa có cái list nào để tấn công
	- cổng 80: chạy nginx và chúng ta có thêm DNS 2million.htb

Chúng ta lưu vào host
![[Pasted image 20261016231555.png]]


# Web enumeration

Trang web
![[Pasted image 20261016231703.png]]



Trong join
![[Pasted image 20261016231725.png]]

- Chúng ta không có invite code nên chưa làm gì được




Source code có code khả nghi ở file inviteapi.min.js

```js
eval(function(p, a, c, k, e, d) {
    e = function(c) {
        return c.toString(36)
    }
    ;
    if (!''.replace(/^/, String)) {
        while (c--) {
            d[c.toString(a)] = k[c] || c.toString(a)
        }
        k = [function(e) {
            return d[e]
        }
        ];
        e = function() {
            return '\\w+'
        }
        ;
        c = 1
    }
    ;while (c--) {
        if (k[c]) {
            p = p.replace(new RegExp('\\b' + e(c) + '\\b','g'), k[c])
        }
    }
    return p
}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}', 24, 24, 'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'), 0, {}))

```


- Ta thấy dòng cuối cùng bị Obfusuction => phân tích nó ta được

```js
function verifyInviteCode(code) { var formData = { "code": code }; $.ajax({ type: "POST", dataType: "json", data: formData, url: '/api/v1/invite/verify', success: function(response) { console.log(response) }, error: function(response) { console.log(response) } }) } function makeInviteCode() { $.ajax({ type: "POST", dataType: "json", url: '/api/v1/invite/how/to/generate', success: function(response) { console.log(response) }, error: function(response) { console.log(response) } }) }
```


- Chúng ta có API
	-  /api/v1/invite/how/to/generate: chắc là để generate
	- /api/v1/invite/verify: để xác minh code

thử POST /api/v1/invite/how/to/generate ta được 1 đoạn tin nhắn bị mã hóa ROT13
![[Pasted image 20261016232550.png]]


Giải mã hóa
![[Pasted image 20261016232711.png]]


- Chúng ta có thêm API /api/v1/invite/generate

Thử API đó ta được invite code bị base64 encrypted

![[Pasted image 20261016232825.png]]

và nó ra được invite code
![[Pasted image 20261016232951.png]]


Xác minh invite code bằng API verify

**![[Pasted image 20261016233122.png]]![[Pasted image 20261016233122 1.png]]


code đã valid

khi đánh vào trang đó thì mình sẽ mở được trang register

![[Pasted image 20261016233206.png]]


Chúng ta register credential test:test

![[Pasted image 20261016234433.png]]



khi ta hit /api/v1
![[Pasted image 20261016234520.png]]

chúng ta có được cách sử dụng api...và chúng ta có API PUT khá lạ


Sau 1 hồi lăn lộn thì chúng ta cũng có thể chỉnh được quyền admin => IDOR api abuse

![[Pasted image 20261016235124.png]]


có vẻ như chúng ta đã tìm được endpoint code injection rồi

![[Pasted image 20261016235546.png]]


# shell as www-data

sử dụng code injection ở trên ta mở python server và inject code của mình lên đó sẽ được shell của www-data

![[Pasted image 20261017000127.png]]
![[Pasted image 20261017000050.png]]


![[Pasted image 20261017000143.png]]


# shell as admin

Chúng ta enumerate hệ thống thì phát hiện credential được lưu ở file .env

![[Pasted image 20261017000259.png]]

creds: admin:SuperDuperPass123



# Shell as root


![[Pasted image 20261017001614.png]]

khi tôi enumerate hệ thống trên thì tôi phát hiện mail và nói bị lỗ hổng Overlays tức CVE-2023-0386

Bằng cách sử dụng repo này https://github.com/puckiestyle/CVE-2023-0386 

Đây là Một lỗ hổng đã được phát hiện trong nhân Linux (Linux kernel), cụ thể là trong hệ thống con OverlayFS, liên quan đến việc truy cập trái phép vào quá trình thực thi các tệp tin có thiết lập quyền đặc biệt (setuid file) đi kèm với các khả năng (capabilities). Lỗ hổng này phát sinh khi người dùng sao chép một tệp tin có chứa các quyền đặc biệt từ một điểm gắn (mount point) được cấu hình là "nosuid" sang một điểm gắn khác. Lỗi ánh xạ định danh người dùng (UID mapping) này cho phép một người dùng cục bộ leo thang đặc quyền trên hệ thống.

![[Pasted image 20261017001839.png]]


thế là về cơ bản chúng ta có root


# Remediation


- Cập nhật Kernel của Host lên cao hơn để vá các lỗ hổng cấp kernel

- Phân chia quyền API hợp lý và tránh việc IDOR sử dụng endpoint api trái phép

- Đừng có giấu code thật ở front-end

- Sử dụng luật và filter để tránh code injection tại các endpoints, thiết lập tường lửa đánh chặn, lọc các input độc hại


# Unintended vector

Thực tế ra bài này có cách khác: từ www-data sau khi làm bước trên chúng ta chỉ cần vector duy nhất

root: GLIBC library 2.35 => CVE-2023-4911 Looney Tunables => root


sử dụng POC này https://github.com/NishanthAnand21/CVE-2023-4911-PoC bạn chỉ cần làm theo các bước và sẽ có quyền root


Bằng chứng exploitable
![[Pasted image 20261028125240.png]]


![[Pasted image 20261028125754.png]]

các bước làm bao gồm
- tải file exploit.c và genlib.py lên máy mục tiêu
- gcc exploit.c -o ten_binary
- python3 genlib.py
- ./ten_binary


và..... tôi có root

![[Pasted image 20261028131830.png]]