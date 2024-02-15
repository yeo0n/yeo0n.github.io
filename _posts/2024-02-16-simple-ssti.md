---
layout: single
title: "simple-ssti 풀이"
date: 2024-02-16 02:27 +0900
categories: 
    - WEB-write-up
tag:
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---

## 문제 정보

존재하지 않는 페이지 방문시 404 에러를 출력하는 서비스입니다.
SSTI 취약점을 이용해 플래그를 획득하세요. 플래그는 flag.txt, FLAG 변수에 있습니다.



## 풀이 

### 소스 코드

```python
#!/usr/bin/python3
from flask import Flask, request, render_template, render_template_string, make_response, redirect, url_for
import socket

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

app.secret_key = FLAG


@app.route('/')
def index():
    return render_template('index.html')

@app.errorhandler(404)
def Error404(e):
    template = '''
    <div class="center">
        <h1>Page Not Found.</h1>
        <h3>%s</h3>
    </div>
''' % (request.path)
    return render_template_string(template), 404

app.run(host='0.0.0.0', port=8000)


```



문제 사이트에 접속하면 404에러 페이지와 robots.txt가 있다.

![image-20240216022758499](/images/2024-02-16-simple-ssti/image-20240216022758499.png)

<br>

둘 다 눌러보면 페이지를 찾을 수 없다고 나온다.

![image-20240216023035125](/images/2024-02-16-simple-ssti/image-20240216023035125.png)


![image-20240216023046888](/images/2024-02-16-simple-ssti/image-20240216023046888.png)

<br>

소스 코드중 Error404라는 함수를 보면 위에 app.errorhandler(404)를 사용해 404 오류가 발생했을 때 이후에 나오는 함수를 호출하는 것을 알 수 있다. 또한 %s 부분에 request.path의 값이 들어가는데 % (request.path)는 Jinja2 템플릿 엔진에서 사용되는 문법인 것을 알 수 있다. 따라서 request.path 부분에 {{5+5}}로 SSTI 취약점을 테스트해볼 수 있다.

```python
@app.errorhandler(404)
def Error404(e):
    template = '''
    <div class="center">
        <h1>Page Not Found.</h1>
        <h3>%s</h3>
    </div>
''' % (request.path)
    return render_template_string(template), 404
```

<br>

아래와 같이 `{{5+5}}`를 요청 경로로 포함하니 5+5가 더해진 10이 출력되는 것을 볼 수 있다.

![image-20240216023810911](/images/2024-02-16-simple-ssti/image-20240216023810911.png)

<br>

flag가 저장된 소스 코드를 분석해보면 app.secret_key에 FLAG를 저장하는 것을 볼 수 있다. 이는 SSTI 취약점을 이용하여 `{{config['SECRET_KEY']}}`를 통해서 FLAG를 출력할 수 있다.  FLASK는 config 객체를 통해서 설정 정보를 저장할 수 있는데 이때 설정 정보의 키가 대문자로 지정되어야 하기 때문에 대문자로 작성하여야 한다.

```python
try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

app.secret_key = FLAG
```



```text
http://host3.dreamhack.games:11484/%7B%7Bconfig[%27SECRET_KEY%27]%7D%7D
```

![image-20240216024753401](/images/2024-02-16-simple-ssti/image-20240216024753401.png)

<br>
아래와 같이 `{{config.items()}}`를 이용해서도 SECRET_KEY에 저장된 FLAG 값을 볼 수 있다.

![image-20240216024923316](/images/2024-02-16-simple-ssti/image-20240216024923316.png)