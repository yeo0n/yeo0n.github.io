---
layout: single
title: "[Lord of SQLInjection] nightmare write-up"
date: 2025-04-28 21:53 +0900
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

## Lord of SQLInjection (nightmare)

### ✏️ 풀이

pw 파라미터에 대해서 필터링 되는 문자를 살펴보면 아래와 같고, pw 파라미터 길이가 6바이트 보다 커도 필터링이 걸리는 걸 알 수 있다.

```
prob _ . () # -
```

![image-20250426165233200](/images/2025-04-26-los-nightmare/image-20250426165233200.png)

<br>

필터링 되는 문자에서 `-`와 `#` 문자가 있으므로 다른 주석 우회 방법을 찾아보니 `;%00` 으로 세미콜론 뒤에 널바이트를 삽입해서 주석을 사용할 수 있다는 것을 확인했다. 또한 입력한 값이 ('') 안에 들어가므로 ') 로 탈출한 후에 0과 비교하면 참이되므로 페이로드는 아래와 같다.

```
')=0;%00
```

![image-20250428215637589](/images/2025-04-26-los-nightmare/image-20250428215637589.png)

