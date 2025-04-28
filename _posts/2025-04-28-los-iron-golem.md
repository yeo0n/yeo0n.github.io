---
layout: single
title: "[Lord of SQLInjection] iron_golem write-up"
date: 2025-04-28 23:30 +0900
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

## Lord of SQLInjection (iron_golem)

### 풀이

이번 문제에서 pw 파라미터에 대해 필터링 되는 문자는 `prob _ . () sleep benchmark` 와 같다. 지난 문제들과 다른 점은 exit 함수로 sql 에러 페이지를 보여주고, result[id] 부분을 쿼리 결과 값으로 echo 해주지 않는다.

![image-20250428233154058](/images/2025-04-28-los-iron-golem/image-20250428233154058.png)

<br>

mysql 에서는 error based sqli를 할 때 서칭을 해보니 `extractvalue`  함수를 사용해서 두 번째 인자에 XPATH가 아니면 이를 쿼리를 실행해서 출력해주는 것 같다. 아래와 같이 페이로드를 삽입하면 extractvalue의 두 번째 인자인 'hi'가 XPATH syntax error로 출력됐다.

```
%27%20or%20extractvalue(%271%27,%20concat(0x3a,%27hi%27))--%20-
```

![image-20250428235557577](/images/2025-04-28-los-iron-golem/image-20250428235557577.png)

<br>

그래서 저 부분을 select pw from prob_iron_golem where id='admin'으로 바로 출력되게 하려 했는데 prob와 _ 문자가 필터링 걸려 있다.