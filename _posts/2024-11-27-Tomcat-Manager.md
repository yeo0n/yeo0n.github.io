---
layout: single
title: "[Dreamhack] Tomcat Manager write-up"
date: 2024-11-27 23:20 +0900
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

## [Dreamhack] Tomcat Manager

<br>

### 문제 설명

- 드림이가 톰캣 서버로 개발을 시작하였습니다.
- 서비스의 취약점을 찾아 플래그를 획득하세요.
- 플래그는 `/flag` 경로에 있습니다.

<br>

### 풀이

문제 사이트에 접속하면 아래와 같은 이미지가 보인다.

<img src="/images/2024-11-27-Tomcat-manager/{8068C30E-9E6C-4E52-9721-B5BFD0F72273}.png" alt="{8068C30E-9E6C-4E52-9721-B5BFD0F72273}" style="zoom: 50%;" />

<br>

페이지 소스코드에서 확인해보면 img 태그로 현재 경로의 `image.jsp` 파일을 이용해 `file` 파라미터로 working.png 파일을 불러오고 있다.

```html
<html>
    <body>
        <center>
            <h2>Under Construction</h2>
            <p>Coming Soon...</p>
            <img src="./image.jsp?file=working.png"/>
        </center>
    </body>
</html>
```

<br>

문제 파일을 받아보면 ROOT.war 이라는 파일과 tomcat-users.xml 이 존재하고 xml 파일은 아래와 같다. tomcat-users.xml 파일은 톰캣을 처음 설치하고 관리자 메뉴를 이용하기 위해서 아래와 같이 `username`과 `password` 부분에 아이디와 비밀번호를 입력해주어야 한다. 보면 id는 `tomcat` 이고 비밀번호는 아직 알지 못하는 것 같다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">

    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <role rolename="manager-jmx"/>
    <role rolename="manager-status"/>
    <role rolename="admin-gui"/>
    <role rolename="admin-script"/>
    <user username="tomcat" password="[**SECRET**]" roles="manager-gui,manager-script,manager-jmx,manager-status,admin-gui,admin-script" />  
</tomcat-users>
```

<br>

burp로 이미지를 GET 메소드로 받아오는 부분을 잡아보면 일단 매우 수상해보인다.. 오른쪽 응답 패킷을 보면 Content-Type에 image/jpeg; 말고 charset=ISO-8859-1이 더 존재한다. gpt에 물어보니 텍스트 데이터를 다루는 파일에 적용된다고 한다. txt, xml, js, html 등이 있다. 

![{40332243-42A8-40B8-B696-C4A2841FB8BC}](/images/2024-11-27-Tomcat-manager/{40332243-42A8-40B8-B696-C4A2841FB8BC}.png)

<br>

우선 file 파라미터 값으로 다른 파일을 받아올 수 있을 것 같다고 생각했고, Dockerfile을 보면 tomcat-users.xml 파일의 경로가 나온다.

![{3EFDE539-E739-4123-85D8-EBCA554CA878}](/images/2024-11-27-Tomcat-manager/{3EFDE539-E739-4123-85D8-EBCA554CA878}.png)

<br>

`../` 문자를 이용해 tomcat-users.xml 파일을 보면 password 값이 평문으로 노출된 것을 확인할 수 있다.

![{7C5515A9-FC1A-406A-A115-80919479BD67}](/images/2024-11-27-Tomcat-manager/{7C5515A9-FC1A-406A-A115-80919479BD67}.png)

<br>

이제 톰캣 관리자 페이지에 들어가기 위해서 `/manager` 엔드포인트에 접속해서 아이디와 비밀번호를 tomcat과 위에서 얻은 비밀번호를 입력해주어야 하는데 왜 안들어가질까... 일단 이 문제는 여기까지 해놔야겠다.

![{3EFACCFD-50CC-4B6C-BB56-43FE54E628FE}](/images/2024-11-27-Tomcat-manager/{3EFACCFD-50CC-4B6C-BB56-43FE54E628FE}.png)