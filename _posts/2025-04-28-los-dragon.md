---
layout: single
title: "[Lord of SQLInjection] dragon write-up"
date: 2025-04-28 23:13 +0900
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

## Lord of SQLInjection (dragon)

### 풀이

이번 문제에서는 필터링 되는 문자는 `prob _ . ()` 밖에 없고, id 값이 admin이 나오면 된다. 근데 입력되는 쿼리를 살펴보면 pw 파라미터 넣는 값 앞에 주석 `#` 이 있어 이를 우회해야 한다.

![image-20250428231353402](/images/2025-04-28-los-dragon/image-20250428231353402.png)

<br>

이번 문제에서는 주석을 어떻게 우회할 수 있는지 묻는 건데, `#` 주석은 뒤에 있는 줄을 무시하는 것이기 때문에, 이를 %0a 를 이용해 줄바꿈 하면 다시 쿼리를 이어서 붙일 수 있다. 그럼 `#and pw'` 가 무시되기 때문에 다시 `and pw=''` 로 guest값이 안나오게 하고 id가 admin인 조건을 추가하면 solve!

```
%27%0aand%20pw=%27%27%20or%20id=%27admin%27--%20-
```

![image-20250428232552271](/images/2025-04-28-los-dragon/image-20250428232552271.png)