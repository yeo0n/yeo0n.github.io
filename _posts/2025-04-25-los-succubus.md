---
layout: single
title: "[Lord of SQLInjection] succubus write-up"
date: 2025-04-25 22:44 +0900
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

## Lord of SQLInjection (succubus)

### 풀이

이번 문제에서는 id와 pw를 필터링 하는 문자가 같다.

![image-20250425224442849](/images/2025-04-25-los-succubus/image-20250425224442849.png)

처음으로는 아래에 해당하는 문자들을 막고 두 번째는 싱글쿼터(')가 들어가면 HeHe로 필터링하고 있다. 

```
prob
_
.
()
```

<br>

이번 문제는 싱글쿼터 우회에 대한 얘기인데 쿼리 안에서 싱글쿼터 앞에 역슬래쉬(`\`)가 들어가면 뒤에 있는 싱글쿼터가 문자로 인식하기 떄문에 마지막 pw에 들어가는 값은 문자열을 빠져나와 or 1=1-- - 만 해주면 우회할 수 있다. 따라서 id는 `\` 만 입력하고 pw는 or 1=1-- - 를 넣어주면 solve!

![image-20250425230133402](/images/2025-04-25-los-succubus/image-20250425230133402.png)

