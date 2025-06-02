---
layout: single
title: "[Dreamhack] insane python write-up"
date: 2025-05-31 11:43 +0900
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

## [Dreamhack] insane python

### 문제 설명

I love Python, but I hate Python because it has secrets.
:(

<br>

### ✏️ 풀이

문제에 접속하면 빈페이지가 반환된다.

![image-20250529230923238](/images/2025-05-29-insane-python/image-20250529230923238.png)

<br>

index.html 소스 코드를 보면 strong 태그 안에 value 값을 받아오고 있다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
</head>
<body>
    <strong> {{ value }} </strong>
</body>
</html>
```

<br>

app.py 소스 코드를 분석해보면, CONFIG 변수에 SECRET 값으로 flag 값이 저장되어 있고, DataConfig 클래스에는 config_data 라는 변수의 인스턴스로 사용된다. `/` 경로에서는 body 파라미터를 받아와서 value에 넘겨주게 되는데, 넘겨줄 때 print_data 함수를 보면 config_data의 포맷 스트링 형태로 넘겨줘야 한다는 것을 알 수 있다.

```python
from flask import Flask, render_template, request
import json
import sys
import io

app = Flask(__name__)

CONFIG = {
 "SECRET": "DH{fake-fake-fake}"
}

class DataConfig(object):
    def __init__(self, name):
        self.name = name

def print_data(format_string, config_data):
    return format_string.format(config_data=config_data)
     
config_data = DataConfig("How can I get the flag?")

@app.route('/',methods=['GET'])
def page1():    
    payload = request.args.get('body')
    sys.stdin = io.StringIO(payload)
    result = print_data(sys.stdin.readline(), config_data)
    return render_template('index.html', value=result)

if __name__ == '__main__':
    app.run(host='0.0.0.0',port=8080)
```

<br>

`print_data` 함수에서 return 문으로 format을 지정할 때 config_data로 지정하였기 때문에 body 파라미터에 `{config_data}`  값을 넣어서 요청해보면 아래와 같이 DataConfig 객체가 출력된다.

![image-20250531112446596](/images/2025-05-29-insane-python/image-20250531112446596.png)

<br>

이번에는 `{config_data.name}` 으로 요청해보면 How 문자열이 잘 출력되는 걸 볼 수 있는데 파이썬의 format 함수의 특징으로는 name 속성을 가져와서 보여주기 때문이다. 하지만 파이썬의 format 함수는 name 속성 뿐만 아니라 객체 내부의 전역 변수에도 접근할 수 있고, 사용자 입력을 따로 필터링하지 않기 때문에 SSTI 취약점이 존재한다고 볼 수 있다.

![image-20250531113519650](/images/2025-05-29-insane-python/image-20250531113519650.png)

<br>

따라서 `{config_data.__init__.__globals__}` 값을 body 파라미터에 넘겨주면 전역 변수를 볼 수 있는데, 우리가 원하는 CONFIG의 SECRET 값으로 flag가 잘 보이는 것을 알 수 있다. flag 값만 가져오기 위해서는 뒤에 [CONFIG][SECRET]을 붙여주면 flag 값만 가져올 수 있다.

![image-20250531114111233](/images/2025-05-29-insane-python/image-20250531114111233.png)

![image-20250531114028219](/images/2025-05-29-insane-python/image-20250531114028219.png)











