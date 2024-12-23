---
layout: single
title: "[Dreamhack] simple-ssti write-up"
date: 2024-12-16 22:00 +0900
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

## [Dreamhack] simple-ssti 

<br>

### 문제 설명

- 존재하지 않는 페이지 방문시 404 에러를 출력하는 서비스입니다.
- SSTI 취약점을 이용해 플래그를 획득하세요. 플래그는 flag.txt, FLAG 변수에 있습니다.

<br>

### 풀이

문제를 살펴보면 path 부분인 경로를 받아와서 404 에러 페이지에서 %s 포맷 스트링을 이용해서 페이지에 그대로 출력해주고 있다.

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

<br>

python의 jinja2 문법으로 path에 {{5*5}}를 입력하면 아래와 같이 5 * 5가 계산된 25가 페이지에 연산되어 출력된 것을 알 수 있다.

![{DA5E997A-6ED6-47A9-BBBB-683DE7EF4F48}](/images/2024-12-23-simple-ssti/{DA5E997A-6ED6-47A9-BBBB-683DE7EF4F48}.png)

<br>

따라서 path에 {{config.items()}}를 입력하면 아래와 같이 쉽게 flag를 획득할 수 있다.

![{9DED70C1-E124-4033-A03F-4D37A4780BF2}](/images/2024-12-23-simple-ssti/{9DED70C1-E124-4033-A03F-4D37A4780BF2}.png)
