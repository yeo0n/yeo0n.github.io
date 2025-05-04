---
layout: single
title: "[Lord of SQLInjection] green_dragon write-up"
date: 2025-05-03 20:30 +0900
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

## Lord of SQLInjection (green_dragon)

### 풀이

이번 문제를 살펴보면 id 와 pw에 키워드 되는 필터링은 `prob _ . ' "` 문자가 같이 필터링되고 있다. 그 다음 부분을 보면 쿼리 결과에서 똑같은 키워드로 한 번 더 필터링을 거치고 id값이 admin이면 solve이다.

![image-20250503205724818](/images/2025-05-03-los-green_dragon/image-20250503205724818.png)

<br>

일단 가장 먼저 \ 문자를 필터링하지 않아서 싱글쿼터 앞에 \ 문자가 있으면 해당 싱글쿼터가 문자열로 인식된다. 따라서 아래와 같이 페이로드를 짜봐서 넣어보면 echo로 query2라는 값이 출력되지 않는 것으로 봐서, 애초에 prob_green_dragon 테이블에 데이터가 없다는 것을 알 수 있다.

![image-20250503212214246](/images/2025-05-03-los-green_dragon/image-20250503212214246.png)

<br>

union 필터링이 안되어 있으므로, 컬럼수를 맞춰주기 위해서 1,2로 테스트해보면 아래와 같이 query2가 잘 뜨는 것을 알 수 있다.

![image-20250503212416939](/images/2025-05-03-los-green_dragon/image-20250503212416939.png)

<br>

1 대신에 \ 문자를 넣어보니 query2가 출력되지 않는다. 따라서 hex 값으로 치환해줘야 하는데 query2 쪽에 들어가는 것도 \문자로 id 조건의 싱글쿼터를 무시하고 union 키워드를 동일하게 이용해서 select char(admin) 으로 만들어주면 solve

![image-20250504171948528](/images/2025-05-03-los-green_dragon/image-20250504171948528.png)

<br>

```
id=\&pw=%20union%20select%200x5c,0x756e696f6e2073656c65637420636861722839372c3130302c3130392c3130352c313130292d2d202d--%20-
```

![image-20250504172604587](/images/2025-05-03-los-green_dragon/image-20250504172604587.png)
