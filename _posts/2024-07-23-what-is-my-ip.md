---
layout: single
title: "[Dreamhack] what-is-my-ip write-up"
date: 2024-07-23 22:45 +0900
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

## [Dreamhack] what-is-my-ip

<br>

### 문제 설명

- How are they aware of us even behind the wall?

<br>

### 풀이

문제 페이지에 접속하면 먼저 내 외부 IP 주소가 나온다. 

![스크린샷 2024-07-23 오후 10.18.04](/images/2024-07-23-what-is-my-ip/스크린샷 2024-07-23 오후 10.18.04.png)

<br>

문제의 소스 코드 부분은 아래와 같고 user_ip라는 변수를 bash쉘을 통해 echo로 보여주고 있다. 여기서 reqeust의 access_route 부분은 처음 들어보았다.

```python
@app.route('/')
def flag():
    user_ip = request.access_route[0] if request.access_route else request.remote_addr
    try:
        result = run(
            ["/bin/bash", "-c", f"echo {user_ip}"],
            capture_output=True,
            text=True,
            timeout=3,
        )
        return render_template("ip.html", result=result.stdout)

    except TimeoutExpired:
        return render_template("ip.html", result="Timeout!")
```

<br>
request의 `access_route` 함수에 대해서 찾아보니 Flask와 같은 웹 프레임워크에서 사용되는 속성으로, 클라이언트가 서버에 접속할 때 거치는 IP 주소들의 리스트를 제공한다고 한다. 또한 주로 프록시 서버를 거쳐 오는 클라이언트의 원래 IP를 추적하는데 사용한다고 되어있다.

<br>

`access_route`의 작동방식은 요청 헤더의 X-Forwarded-For 헤더를 기반으로 작동하며, Flask에서 이 헤더의 요청을 추가적으로 검증하지 않으면 요청 헤더를 추가하여 이를 변조할 수 있다. 따라서 `access_route` 는 X-Forwarded-For 헤더에 저장된 ip들이고 인덱스 0번째의 값이 첫 ip 주소이기 때문에 이를 가져오고, 위 코드와 같이 이 헤더가 존재하지 않으면 `remote_addr`에서 가져온다. 

<br>

이는 페이지를 새로고침했을 때의 패킷을 Burp Suite를 통해 잡은 것이다.

![image-20240723223851570](/images/2024-07-23-what-is-my-ip/image-20240723223851570.png)

<br>

그럼 X-Forwarded-For 요청 헤더를 요청 패킷 아래에 추가하고 이는 Command Injection을 통해 `; cat /flag` 문자열을 추가해주면 응답값과 같이  flag를 읽어온 것을 확인할 수 있다.

![스크린샷 2024-07-23 오후 10.40.16](/images/2024-07-23-what-is-my-ip/스크린샷 2024-07-23 오후 10.40.16.png)