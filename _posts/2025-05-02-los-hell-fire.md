---
layout: single
title: "[Lord of SQLInjection] hell_fire write-up"
date: 2025-05-02 01:39 +0900
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

## Lord of SQLInjection (hell_fire)

### 풀이

이번 문제는 `prob _ . proc union` 키워드를 필터링하고, order 파라미터를 넣어 요청할 때 result의 id값이 admin이 있으면 email을 `*` 문자로 표시하고 있다. 그리고 아래 부분을 보면 `addslashes` 함수로 email 파라미터를 받아와 필터링하기 때문에 admin의 email 값을 알아내야 solve 할 수 있다.

![image-20250502013921437](/images/2025-05-02-los-hell-fire/image-20250502013921437.png)

<br>
