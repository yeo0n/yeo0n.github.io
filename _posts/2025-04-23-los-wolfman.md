---
layout: single
title: "[Lord of SQLInjection] wolfman write-up"
date: 2025-04-23 00:39 +0900
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

## Lord of SQLInjection (wolfman)

### ✏️ 풀이

문제를 살펴보면 전 orc 문제와 동일하게 `prob _ . ()` 문자를 필터링하고, 추가적으로 공백까지 더해서 필터링하고 있다. 쿼리에서는 guest로 되어있는데 result 값 id가 admin이면 solve이다.

![image-20250423004035677](/images/2025-04-23-los-wolfman/image-20250423004035677.png)

<br>

pw 파라미터에`'%0a1=1--%0a-'` 값을 넣어 공백을 개행 문자인`\n`로 url 인코딩한 `%0a` 를 넣어 테스트해보면 Hello guest가 잘 출력되는 것을 확인할 수 있다.

![image-20250423004803495](/images/2025-04-23-los-wolfman/image-20250423004803495.png)

<br>

해당 문제에서 처음에 union select 'admin'을 시도해보려다 연산자 우선순위 문제인 것을 알았다. SQL 쿼리에서 and 연산자는 or 연산자보다 먼저 처리되기 때문에 쿼리가 아래와 같이 들어가면 `(id=guest and pw=1) or (id=admin and 1=1)` 처럼 쿼리가 인식한다. 따라서 앞에 id가 guest인 조건은 pw가 만족하지 못하므로 뒤에 있는 id가 admin인 조건이 참이되고 result['id']가 admin이 되는 것이다.

```
1%27%0aor%0aid=%27admin%27%0aand%0a1=1--%0a-%27
```

![image-20250423011003771](/images/2025-04-23-los-wolfman/image-20250423011003771.png)
