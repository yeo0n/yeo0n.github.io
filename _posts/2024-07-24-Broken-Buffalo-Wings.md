---
layout: single
title: "[Dreamhack] Broken Buffalo Wings write-up"
date: 2024-07-24 22:33 +0900
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

## [Dreamhack] Broken Buffalo Wings

<br>

### 문제 설명

### Description

Buffalo wings, also known as hot wings or chicken wings...

### Notice

해당 문제는 의도된 풀이가 존재하지만 다른 쉬운 방법으로도 문제를 해결할 수 있습니다!
패치된 버전의 문제에서 제공되는 파일과 해당 문제 파일의 diff를 확인해 보세요.

<br>

### 풀이

문제 서버로 접속해보면 버팔로 윙 이미지가 ChatGPT로 만들어져있고 아래는 Comment를 입력하는 부분이 존재한다.

<img src="/images/2024-07-24-Broken-Buffalo-Wings/image-20240724223848475.png" alt="image-20240724223848475" style="zoom: 33%;" />

<br>

소스 코드를 살펴보면 index.php와 report.php가 존재하는데 report.php로 요청하면 아래와 같은 아마 관리자의 계정으로 key=value 형태로 요청을 수행하는 폼이 존재하는 것 같다.

![image-20240724224213991](/images/2024-07-24-Broken-Buffalo-Wings/image-20240724224213991.png)

<br>

아래는  report.php의 소스 코드인데 ?? 연산자는 널 병합 연산자로 7.0 버전부터 생겼다고 하며 값이 존재하지 않을 경우 오른 쪽의 haha가 삽입된다. `escapeshellarg` 함수는 문자열 주위에 작은 따옴표를 추가하고 기존의 작은 따옴표를 이스케이프하여 안전한 인수로 처리하는 함수이다. 이를 bot.py를 이용하여 쉘 명령을 실행하고 있다. 

![image-20240724224811073](/images/2024-07-24-Broken-Buffalo-Wings/image-20240724224811073.png)

<br>
아래는 bot.py의 소스 코드인데  /flag.txt 에 flag가 저장되어 있고, 파라미터와 쿠키를 받아오고, 2번째 인덱스가 /라면 /를 제거하는 것을 알 수 있다. 

```python
try:
    FLAG = open("./flag.txt", "r").read()
except:
    FLAG = "DH{{This_is_flag}}"


def read_url(paramters, cookie={"name": "name", "value": "value"}):
    parameter = ""

    if paramters[1][0] == "/":
        parameter = paramters[1][1:]
    else : 
        parameter = paramters[1]

    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "--headless",
            "--window-size=1920x1080",
            "--disable-gpu",
            "--no-sandbox",
            "--disable-dev-shm-usage",
        ]:
            options.add_argument(_)

        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")

        driver.add_cookie(cookie)
        driver.get(f"http://127.0.0.1:8000/{parameter}")
    except Exception as e:
        driver.quit()
        return False
    driver.quit()
    return True


read_url(argv, {"name": "flag", "value": FLAG})

```

<br>
다시 위로 올라가서 문제 설명을 잘 읽어보면 원래의 buffalo wings의 문제와 파일이 다르다고 되어있다. 확인해보니 아래가 원래의  buffalo wings의 nginx.conf 파일의 소스 코드인데 /flag.txt 의 경로가 403으로 제한되어 있다.

![image-20240724231010482](/images/2024-07-24-Broken-Buffalo-Wings/image-20240724231010482.png)

<br>

아래는 broken buffalo의 nginx.conf 파일의 소스 코드인데 /flag.txt 경로를 제한하는 소스 코드가 없는 것을 확인할 수 있었다. 

![image-20240724231201755](/images/2024-07-24-Broken-Buffalo-Wings/image-20240724231201755.png)

<br>

/flag.txt 경로로 요청해보면 아래와 같이 flag 값이 나온 것을 확인할 수 있다..

![image-20240724231501851](/images/2024-07-24-Broken-Buffalo-Wings/image-20240724231501851.png)