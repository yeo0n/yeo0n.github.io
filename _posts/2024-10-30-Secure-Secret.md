---
layout: single
title: "[Dreamhack] Secure Secret write-up"
date: 2024-10-30 18:30 +0900
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

## [Dreamhack] Secure Secret

<br>

### 문제 설명

- 플래그 파일을 무작위한 디렉터리에 배치한 후 아무도 모르게끔 그 경로를 세션에 숨겨두었습니다.

- 문제점을 찾고 익스플로잇하여 플래그를 획득하세요!

- 플래그 형식은 DH{...} 입니다.

<br>

### 풀이

문제 파일을 받아서 소스 코드를 확인해보면 다음과 같다.

```python
#!/usr/bin/env python3
import os
import string
from flask import Flask, request, abort, render_template, session

SECRETS_PATH = 'secrets/'
ALLOWED_CHARACTERS = string.ascii_letters + string.digits + '/'

app = Flask(__name__)
app.secret_key = os.urandom(32)

# create sample file
with open(f'{SECRETS_PATH}/sample', 'w') as f:
    f.write('Hello, world :)')

# create flag file
flag_dir = SECRETS_PATH + os.urandom(32).hex()
os.mkdir(flag_dir)
flag_path = flag_dir + '/flag'
with open('/flag', 'r') as f0, open(flag_path, 'w') as f1:
    f1.write(f0.read())


@app.route('/', methods=['GET'])
def get_index():
    # safely save the secret into session data
    session['secret'] = flag_path

    # provide file read functionality
    path = request.args.get('path')
    if not isinstance(path, str) or path == '':
        return render_template('index.html', msg='input the path!')

    if any(ch not in ALLOWED_CHARACTERS for ch in path):
        return render_template('index.html', msg='invalid path!')

    full_path = f'./{SECRETS_PATH}{path}'
    if not os.path.isfile(full_path):
        return render_template('index.html', msg='invalid path!')

    try:
        with open(full_path, 'r') as f:
            return render_template('index.html', msg=f.read())
    except:
        abort(500)

```

<br>

가장 먼저 변수부터 살펴보면 `SECRET_PATH`라는 변수에 `secrets/`라는 경로를 정의했고, 아래 `ALLOWED_CHARACTERS` 변수를 보면 `string.ascii_letters` 함수를 사용해서 알파벳 소문자와 대문자, 그리고 `string.digits` 함수로 숫자, `/`문자까지 디렉터리 경로에 허용하는 문자열들을 정의한 것 같다.

```python
SECRETS_PATH = 'secrets/'
ALLOWED_CHARACTERS = string.ascii_letters + string.digits + '/'
```

<br>

주석의  sample file 부분을 보면 위에 `SECRET_PATH` 변수에 secrets/ 라고 되어있으므로, 이 경로에 sample이라는 파일을 생성해서 `Hello,world :)` 텍스트를 작성하는 것을 알 수 있다. 아래 주석의 flag file을 보면 `os.urandom(32)` 함수로 32바이트의 랜덤 문자열을 생성하고, `hex()` 함수로 이를 16진수의 문자열로 변경하고 있다. 따라서 1바이트는 0~255까지 표현할 수 있고 이를 16진수로는 2자리로 표현할 수 있기 때문에 64자리의 문자열이 생성되는 것을 알 수 있다. 그 다음은 경로에 `flag` 파일을 생성해서 flag를 읽어오는 것을 알 수 있다.

```python
# create sample file
with open(f'{SECRETS_PATH}/sample', 'w') as f:
    f.write('Hello, world :)')

# create flag file
flag_dir = SECRETS_PATH + os.urandom(32).hex()
os.mkdir(flag_dir)
flag_path = flag_dir + '/flag'
with open('/flag', 'r') as f0, open(flag_path, 'w') as f1:
    f1.write(f0.read())
```

<br>

아래 코드를 보면 세션 데이터에 `secret`이라는 키로 `flag_path` 를 저장하고, GET 메소드로 `path`라는 파라미터를 받아와서  `ALLOWED_CHARACTERS` 함수로 `../`와 같은 문자를 쓰지 못하도록 위에 허용된 문자들만 사용할 수 있도록 하는 것을 알 수 있다. 그리고 앞에 `{SECRETS_PATH}` 경로를 붙여줘서 path 경로만 우리가 입력해주면 되고, 그 경로에 파일이 존재하지 않으면 `invalid path!` 문자열을 출력해준다. 마지막으로 파일을 열면서 오류가 발생하면 서버 500 에러 코드를 반환한다. 

```python

@app.route('/', methods=['GET'])
def get_index():
    # safely save the secret into session data
    session['secret'] = flag_path

    # provide file read functionality
    path = request.args.get('path')
    if not isinstance(path, str) or path == '':
        return render_template('index.html', msg='input the path!')

    if any(ch not in ALLOWED_CHARACTERS for ch in path):
        return render_template('index.html', msg='invalid path!')

    full_path = f'./{SECRETS_PATH}{path}'
    if not os.path.isfile(full_path):
        return render_template('index.html', msg='invalid path!')

    try:
        with open(full_path, 'r') as f:
            return render_template('index.html', msg=f.read())
    except:
        abort(500)
```

<br>

문제 사이트에 접속해서 입력하게 될 부분은 path 파라미터로 sample이라고 입력하게 되면 `full_path` 상으로 `./secrets/sample` 을 읽어오게 되고 제출해보면 아래와 같이 hello world가 출력되는 것을 알 수 있다. 

<img src="/images/2024-10-30-Secure-Secret/image-20241030194450941.png" alt="image-20241030194450941" style="zoom:50%;" />

<br>

일단 flag를 읽어오기 위해서 정말 여러가지 삽질을 많이 해본 것 같다. 먼저 세션 쿠키 부분이 어떻게 구성되어 있나 알아보니 `[Payload].[Timestamp].[Signature]` 이런 식으로 되어 있고 문제 서버의 Payload 부분은 `eJwtyjEOgCAMQNG7cAHaChS8TUupi5O4Ge-uJiZ_eMO_whz9GGdYf8xoQJXgjRr0ZgjapYiZMruR16LCOSfgWkBGUhgLIpZm_t05Rd9lC_cD1TwadA`와 같고,  `.`으로 구별되면서 Timestamp 부분은 없었다.

<br>

처음엔 `secret_key`를 무조건 알아야 세션 쿠키를 복호화할 수 있다고 생각하고, 디렉토리 리스팅 우회 공격 방법을 막 찾아보았다. 근데 코드 상에서는 이미 알파벳과 숫자, `/` 문자만 허용하고 있었기 때문에 이를 이용해서 뭔가 해보는 것도 쉽지 않았다.

```python
app.secret_key = os.urandom(32)
```

<br>

그래서 다시 이 세션 쿠키 값을 분석해보면서 flask의 세션 쿠키에서 Payload 부분의 값이 직렬화와 base64 URL-safe로 인코딩되어 있다는 것을 알았다. 그래서 웹 페이지에 base64 url-safe로 디코딩을 해보았는데 전혀 알 수 없는 문자가 나오는 것이다. 그래서 아 이건 무조건 `secret_key`를 알아야 복호화를 진행할 수 있구나라고 생각하고 있었다.
![image-20241031200621791](/images/2024-10-30-Secure-Secret/image-20241031200621791.png)

<br>

그러던 찰나, 구글링을 진행해보면서 나와 비슷하게 flask 세션 쿠키를 어떤 스크립트 파일로 돌리니 비밀키 없이도 성공적으로 디코딩이 가능하다는 글을 보게 되었다.

https://stackoverflow.com/questions/77340063/flask-session-cookie-tampering 

![image-20241031201019427](/images/2024-10-30-Secure-Secret/image-20241031201019427.png)

<br>

그래서 바로 이 `flask_sessin_cookie_manager3.py`를 이용해서 세션 쿠키를 디코딩해보니 세션 데이터 안에 있는 경로가 잘 나온 것을 알 수 있다. 근데 이해가 잘 안되었던게 왜 인터넷으로 base64url로 디코딩을 하면 잘 나오지 않았는데 이 파이썬 파일을 사용하니 잘 나오는게 이해가 안됐다. 그래서 gpt에게 물어보면서 세션 쿠키를 base64url로 디코딩을 진행하는데 `secret_key`가 있는 경우는 서명을 검증하는 절차를 거쳐 안전하게 복호화를 진행할 수 있고, `secret_key`가 없는 경우는 서명을 무시하고 base64 url-safe로 디코딩할 수 있는데 무결성이 보장되지 않는다는 점을 알았다. 

![image-20241031201208894](/images/2024-10-30-Secure-Secret/image-20241031201208894.png)

<br>

그런데도 왜 base64 url-safe로 디코딩하였는데도 이상한 문자들로 보였는지 찾아보니 이 값이 추가적으로 압축되어 있을 가능성이 있었고, 이 데이터를 `zlib` 을 통해 압축을 해제할 수 있다는 것을 알게 되었다. 그래서 아래와 같이 위 스크립트를 사용해서 문제를 푸는게 아닌 내가 직접 코드를 작성해서도 문제를 풀 수 있다는 것을 확인하고 싶었다. 아래 코드를 작성하면서 오류가 하나 발생했었다. base64 인코딩은 4바이트씩 데이터를 그룹화하여 인코딩을 진행하기 때문에, 데이터의 길이가 4의 배수가 아니라면 패딩(=) 문자가 필요하다. 따라서 패딩을 추가한 값에 base64 url-safe로 디코딩을 진행하고 zlib으로 압축을 해제하면 디렉터리 경로를 출력할 수 있다.

```python
import base64
import zlib

# Base64 URL-safe 인코딩된 값
encoded_data = "eJwtyjEOgCAMQNG7cAHaChS8TUupi5O4Ge-uJiZ_eMO_whz9GGdYf8xoQJXgjRr0ZgjapYiZMruR16LCOSfgWkBGUhgLIpZm_t05Rd9lC_cD1TwadA"

# 패딩 추가
encoded_data += '=' * (-len(encoded_data) % 4)

# Base64 URL-safe 디코딩
decoded_data = base64.urlsafe_b64decode(encoded_data)

# zlib 압축 해제
try:
    decompressed_data = zlib.decompress(decoded_data)
    print("압축 해제 후 데이터:", decompressed_data)
except zlib.error as e:
    print("압축 해제 오류:", e)
```

![image-20241031202410202](/images/2024-10-30-Secure-Secret/image-20241031202410202.png)

<br>
나온 경로를 문제 서버에 입력하기 위해서 path 파라미터 부분 앞에는 `./secrets/`가 이미 작성되어 있기 때문에 뒤에 있는 부분들만 넣어주면 아래에 flag 값이 잘 나오는 것을 확인할 수 있다. 

![image-20241031202537713](/images/2024-10-30-Secure-Secret/image-20241031202537713.png)
