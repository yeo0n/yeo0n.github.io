---
layout: single
title: "[Lord of SQLInjection] skeleton write-up"
date: 2025-04-23 23:10 +0900
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

## Lord of SQLInjection (skeleton)

### ✏️ 풀이

문제에서 먼저 preg_match 함수를 보면 `prob _ . ()` 문자를 pw 파라미터에 대해서 필터링하고 있다. 그리고 해당 쿼리의 결과 값의 id가 admin이면 solve이다.

![image-20250423231233494](/images/2025-04-23-los-skeleton/image-20250423231233494.png)

<br>

pw 파라미터에 아래 페이로드를 넣어주면, 전의 문제들과 동일하게 and 연산자가 or 보다 먼저 계산되기 때문에 괄호가 쳐져 있다고 생각하면 된다. 마지막으로 뒤에는 주석을 이용해 문제를 해결할 수 있다.

```url
1' or id='admin' and 1=1-- -
```

![image-20250423231553678](/images/2025-04-23-los-skeleton/image-20250423231553678.png)
