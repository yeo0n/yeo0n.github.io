---
layout: single
title: "[Dreamhack] Dreampring write-up"
date: 2025-05-18 16:40 +0900
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

## [Dreamhack] Dreampring

### 문제 설명

- **Dreampring** 서비스의 취약점을 찾아 플래그를 획득하세요.

<br>

### 풀이

문제 소스 코드를 받아보면 war 파일 하나랑 Dockerfile 밖에 존재하지 않는다.

<br>

일단 문제 이름과 war 파일로 보았을때 문제가 spring과 관련된 것 같다. 문제 사이트에 접속해보면 아래와 같다.

![image-20250518164933671](/images/2025-05-18-Dreampring/image-20250518164933671.png)

<br>

계정을 생성하려고 하니 아래와 같이 Username을 정규표현식에 맞춰서 작성해야 한다고 한다.

![image-20250518165428150](/images/2025-05-18-Dreampring/image-20250518165428150.png)

<br>

guest와 비밀번호는 임의로 작성해서 일단 계정을 생성을 해보면 Profile 경로에서 사용자 이름과 설명, 이메일이 출력되고 있다.

![image-20250518165822499](/images/2025-05-18-Dreampring/image-20250518165822499.png)

![image-20250518165836780](/images/2025-05-18-Dreampring/image-20250518165836780.png)

<br>

새로운 계정을 만들어서 특수문자를 테스트해보면 아래와 같이 필터링 처리가 되어서 xss 는 쉽지 않아 보인다.

![image-20250518170724282](/images/2025-05-18-Dreampring/image-20250518170724282.png)

<br>

war 파일을 분석하기 위해 `jar xvf` 명령을 이용해서 war 파일을 풀어주었다.

![image-20250518191502145](/images/2025-05-18-Dreampring/image-20250518191502145.png)

<br>

풀어서 확인해보면 WEB-INF 폴더 아래 jsp 파일들을 확인해볼 수 있다. 여기서 admin과 관련된 jsp 파일들이 존재하고 있다.

![image-20250518191828598](/images/2025-05-18-Dreampring/image-20250518191828598.png)

<br>

/admin 경로로 접속해보니 `you are not admin` 문자열로 관리자가 아니라는 에러 alert 창이 뜨고 있다.

![image-20250518191943760](/images/2025-05-18-Dreampring/image-20250518191943760.png)

<br>

main.jsp 파일을 보면 메인 페이지에서 roleid 값이 1337이라면 /admin 경로를 띄워주는 것을 알 수 있다.

![image-20250518192334204](/images/2025-05-18-Dreampring/image-20250518192334204.png)

<br>

class 파일들은 jadx-gui를 이용해서 소스 코드를 분석할 수 있다. MainApplication을 보면 관리자 이름을 admin으로 지정하여 `PasswordUtil.generate()` 를 이용해 패스워드를 넣고 있다. 또한 `WinError.ERROR_INVALID_SID`가 아마도 1337 값이고, intro 부분에 flag값이 저장되는 것을 알 수 있다.

![image-20250518195534074](/images/2025-05-18-Dreampring/image-20250518195534074.png)

<br>

Services 폴더를 보면 현재 경로의 data 폴더 안에 username.json 파일이 계정을 생성하면 저장된다는 것을 알 수 있다.

![image-20250518205407794](/images/2025-05-18-Dreampring/image-20250518205407794.png)


<br>

HTTPUtil 부분을 보면 X-Forwarded-For 등 헤더에서 IP를 가져오기 때문에 아래 XFF 헤더를 설정해주고, roleId 값을 1337로 설정하여 패킷을 보낸 후 data/guest2.json 파일을 확인해보면 ip와 roleId값이 1337로 잘 설정된 것을 확인할 수 있다. 

![image-20250518205815907](/images/2025-05-18-Dreampring/image-20250518205815907.png)

![image-20250518205830855](/images/2025-05-18-Dreampring/image-20250518205830855.png)

<br>

MainController 부분을 살펴보면, 로그인할 때 roleId 값이 1337이고, 요청하는 IP와 저장된 IP가 127.0.0.1인지 검사하고 있다. 따라서 Burp로 프록시를 잡아 XFF헤더에 127.0.0.1 값을 넣어주면 아래와 같이 Admin 네비게이션 탭이 뜬 것을 확인할 수 있다.

![image-20250518210844271](/images/2025-05-18-Dreampring/image-20250518210844271.png)

![image-20250518210620305](/images/2025-05-18-Dreampring/image-20250518210620305.png)

<br>

그럼 이제 admin 탭에 들어가서 Username 검색 창에 admin을 입력하면 아래 flag 값이 뜨는 것을 확인할 수 있다. Docker에서 확인했으므로 dreamhack 서버에서도 똑같이 수행해주면 되겠다.

![image-20250518211145750](/images/2025-05-18-Dreampring/image-20250518211145750.png)

![image-20250518211619025](/images/2025-05-18-Dreampring/image-20250518211619025.png)