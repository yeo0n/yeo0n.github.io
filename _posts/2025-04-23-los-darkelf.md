---
layout: single
title: "[Lord of SQLInjection] darkelf write-up"
date: 2025-04-21 01:14 +0900
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

## Lord of SQLInjection (darkelf)

### ✏️ 풀이

해당 문제에서는 preg_match 함수로 `prob _ . ()` 문자와 `or and` 문자를 필터링하고 있다. 이 필터링을 우회하여 id값이 admin이면 solve이다.

![image-20250423011455564](/images/2025-04-23-los-darkelf/image-20250423011455564.png)

<br>

이건  wolfman 문제에서 or과 and를 우회할 수 있냐를 묻는 거 같은데 or과 and는 `||`와 `&&`를 이용해서 우회하면 쉽게 풀 수 있다. 한가지 주의해야 할 점은 `&` 문자는 일반적인 URL에서 다음 파라미터 값을 사용하기 위해 쓰는 문자로 URL 인코딩하여 `%26` 으로 대체해서 적어주면 된다.

```
1%27%20||%20id=%27admin%27%20%26%26%201=1--%20-
```

![image-20250423011832067](/images/2025-04-23-los-darkelf/image-20250423011832067.png)