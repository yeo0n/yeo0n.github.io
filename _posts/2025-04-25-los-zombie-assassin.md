---
layout: single
title: "[Lord of SQLInjection] zombie_assassin write-up"
date: 2025-04-25 23:05 +0900
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

## Lord of SQLInjection (zombie_assassin)

### 풀이

이번 문제는 addslashes 함수와 strrev 함수를 이용해서 먼저 id와 pw 파라미터 값을 필터링하고 그 다음 키워드에 계속 나왔던 문자로 필터링하고 있다.

![image-20250425230554167](/images/2025-04-25-los-zombie-assassin/image-20250425230554167.png)

<br>

addslashes 함수를 먼저 우회해야 하는데, 이 함수는 특수문자 앞에 역슬래쉬를 붙여서 출력해준다. 적용되는 특수문자는 `' " \ NULL` 이 해당된다. 이는 \ 문자를 url 인코딩하면 %5c인데, %aa%5c가 되면 한 개의 멀티 바이트로 인식해서 싱글쿼터를 우회할 수 있다. 따라서 %aa%5c%27이 되면 싱글쿼터 문자로 인식될 수 있다.

<br>

문제점은 strrev 함수로 입력한 값을 전부 뒤집어서 출력해준다. 이번 문제에서는 멀티바이트를 사용해서 addslashses 함수를 우회하는 문제는 아니고, strrev 함수 덕분에 더블 쿼터를 입력하면 `"\` 으로 들어가서 뒤에 있는 싱글 쿼터가 문자열로 인식되면서 pw 파라미터의 첫 번째 싱글쿼터까지 전부 문자열로 인식된다. 따라서 아래와 같은 페이로드를 쿼리스트링에 붙여 입력해주면 solve!

```
id="&pw=%20-%20--%201=1%20ro
```

![image-20250426164754787](/images/2025-04-25-los-zombie-assassin/image-20250426164754787.png)