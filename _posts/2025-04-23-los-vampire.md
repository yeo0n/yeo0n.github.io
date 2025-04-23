---
layout: single
title: "[Lord of SQLInjection] vampire write-up"
date: 2025-04-23 23:04 +0900
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

## Lord of SQLInjection (vampire)

### ✏️ 풀이

문제를 살펴보면 싱글쿼터(`'`) 를 "No Hack" 문자열로 필터링하고, id 파라미터를 소문자로 변환시킨다. 다음으로 admin 문자열을 공백으로 치환하여 쿼리에 넣고, 결과 id 값이 admin이면 solve이다.

![image-20250423230524372](/images/2025-04-23-vampire/image-20250423230524372.png) 

<br>

해당 문제는 공백으로 치환할 때에 대한 문제인데 이건 `admadminin` 을 입력해주면 가운데 admin이 공백으로 치환되면서 admin이 쿼리에 들어가게 되는 문제이다.

![image-20250423230841524](/images/2025-04-23-vampire/image-20250423230841524.png)