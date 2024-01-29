---
layout: single
title: "[Dreamhack] pathtraversal write-up"
date: 2024-01-30 00:23:22 +0900
categories: 
    - WEB-write-up
tag: PathTraversal 
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---

## [Dreamhack] pathtraversal write-up

<br>

### 문제 설명

- 사용자의 정보를 조회하는 API 서버입니다.
- Path Traversal 취약점을 이용해 `api/flag` 에 있는 플래그를 획득하세요!
  

### 소스 코드

```python
#!/usr/bin/python3
from flask import Flask, request, render_template, abort
from functools import wraps
import requests
import os, json

users = {
    '0': {
        'userid': 'guest',
        'level': 1,
        'password': 'guest'
    },
    '1': {
        'userid': 'admin',
        'level': 9999,
        'password': 'admin'
    }
}

def internal_api(func):
    @wraps(func)
    def decorated_view(*args, **kwargs):
        if request.remote_addr == '127.0.0.1':
            return func(*args, **kwargs)
        else:
            abort(401)
    return decorated_view

app = Flask(__name__)
app.secret_key = os.urandom(32)
API_HOST = 'http://127.0.0.1:8000'

try:
    FLAG = open('./flag.txt', 'r').read() # Flag is here!!
except:
    FLAG = '[**FLAG**]'

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/get_info', methods=['GET', 'POST'])
def get_info():
    if request.method == 'GET':
        return render_template('get_info.html')
    elif request.method == 'POST':
        userid = request.form.get('userid', '')
        info = requests.get(f'{API_HOST}/api/user/{userid}').text
        return render_template('get_info.html', info=info)

@app.route('/api')
@internal_api
def api():
    return '/user/<uid>, /flag'

@app.route('/api/user/<uid>')
@internal_api
def get_flag(uid):
    try:
        info = users[uid]
    except:
        info = {}
    return json.dumps(info)

@app.route('/api/flag')
@internal_api
def flag():
    return FLAG

application = app # app.run(host='0.0.0.0', port=8000)
# Dockerfile
#     ENTRYPOINT ["uwsgi", "--socket", "0.0.0.0:8000", "--protocol=http", "--threads", "4", "--wsgi-file", "app.py"]
```

<br>

### 풀이

소스코드를 분석해보면 아래의 users 변수에 guest와 admin의 계정이 들어있다.

```python
users = {
    '0': {
        'userid': 'guest',
        'level': 1,
        'password': 'guest'
    },
    '1': {
        'userid': 'admin',
        'level': 9999,
        'password': 'admin'
    }
}
```

<br>

아래 함수를 살펴보면 internal_api 함수를 정의하면서 이 함수는 요청하는 IP가 로컬호스트여야 한다는 것을 알려주고 있다. 또한 API_HOST는 로컬호스트의 8000번 포트를 사용하는 것을 알 수 있다.

```python
def internal_api(func):
    @wraps(func)
    def decorated_view(*args, **kwargs):
        if request.remote_addr == '127.0.0.1':
            return func(*args, **kwargs)
        else:
            abort(401)
    return decorated_view

app = Flask(__name__)
app.secret_key = os.urandom(32)
API_HOST = 'http://127.0.0.1:8000'
```

<br>

다음으로 get_info 엔드포인트 부분이다. 이 엔드포인트에서 POST 요청인 입력 폼에 userid를 입력하여 요청하면 `{API_HOST}/api/user/{userid}` 의 주소에 들어가 text 부분을 읽어온다. 만약 userid 가 guest라면 요청되는 주소는 `127.0.0.1:8000/api/user/guest`인 것이다. api/flag 엔드포인트에는 FLAG를 리턴해주므로 이 주소로 관리자 계정으로 들어가는게 포인트인 것 같다.

```python
@app.route('/get_info', methods=['GET', 'POST'])
def get_info():
    if request.method == 'GET':
        return render_template('get_info.html')
    elif request.method == 'POST':
        userid = request.form.get('userid', '')
        info = requests.get(f'{API_HOST}/api/user/{userid}').text
        return render_template('get_info.html', info=info)

@app.route('/api')
@internal_api
def api():
    return '/user/<uid>, /flag'

@app.route('/api/user/<uid>')
@internal_api
def get_flag(uid):
    try:
        info = users[uid]
    except:
        info = {}
    return json.dumps(info)

@app.route('/api/flag')
@internal_api
def flag():
    return FLAG
```



<br>

Burp Suite를 이용해 get_info에서 POST 요청되는 패킷을 잡아보았다. 패킷에 요청되는 userid가 0은 guest 1은 admin으로 출력되는 것을 알 수 있다. 

![image-20240130005119650](/images/2024-01-30-pathtraversal/image-20240130005119650.png)

<br>

바로 `../` 문자를 userid 변수에 넣어주니 Response 부분에 html이 보인다.

![image-20240130005336096](/images/2024-01-30-pathtraversal/image-20240130005336096.png)

<br>

FLAG의 위치는 아까 소스코드를 분석할 때 `api/flag`의 엔드포인트에서 FLAG를 반환하고 있었다.  지금 현재 위치는 `api/user/{userid}` 의 위치이므로 `../` 으로 하나만 입력해주면 `api/` 의 경로가 된다. 따라서 `../flag`를 입력해주니 바로 FLAG 값이 나온 것을 확인할 수 있다. 
{: .notice--danger}

![image-20240130005517864](/images/2024-01-30-pathtraversal/image-20240130005517864.png)

