---
layout: single
title: "[Dreamhack] web-misconf-1 write-up"
date: 2024-02-02 23:05 +0900
categories: 
    - WEB-write-up
tag: php
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---



## 문제 설명

기본 설정을 사용한 서비스입니다.
로그인한 후 Organization에 플래그를 설정해 놓았습니다.

<br>

## 풀이

접속 정보로 접속하면 아래와 같이 완전 기본 셋팅된 페이지가 보인다. 

![image-20240202231133030](/images/2024-02-02-web-misconf-1/image-20240202231133030.png)

<br>

문제 파일은 `default.ini` 파일이 있다. 이 중에 내려보니 admin 계정이 기본으로 주어졌고 username과 password가 admin, admin인 것을 확인할 수 있다.

```default
# default admin user, created on startup
admin_user = admin

# default admin password, can be changed before first start of grafana, or in profile settings
admin_password = admin
```

<br>

admin으로 로그인하니 dashboard 창이 보인다. 

![image-20240202232558563](/images/2024-02-02-web-misconf-1/image-20240202232558563.png)

<br>

organization에 flag가 있다고 해서 들어가니 일단 flag가 보이지 않는다. 

![image-20240202232911172](/images/2024-02-02-web-misconf-1/image-20240202232911172.png)

<br>

그래서 바로 Settings로 들어가 쭉 내리다 보니 flag를 확인할 수 있었다.

![image-20240202235741938](/images/2024-02-02-web-misconf-1/image-20240202235741938.png)