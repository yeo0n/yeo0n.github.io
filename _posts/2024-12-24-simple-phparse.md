---
layout: single
title: "[Dreamhack] simple-phparse write-up"
date: 2024-12-24 13:14 +0900
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

## [Dreamhack] simple-phparse

<br>

### 문제 설명

- php로 작성된 페이지입니다.
- 플래그는 `/flag.php`에 위치합니다.
- 플래그의 형식은 DH{...} 입니다.

<br>

### 풀이

<br>

문제 사이트에 일단 접속해보자.

![{D456BB55-60F2-4C48-8D58-F0FD3EE6E6BE}](/images/2024-12-24-simple-phparse/{D456BB55-60F2-4C48-8D58-F0FD3EE6E6BE}.png)

<br>

문제 소스 코드를 분석해보면 되게 간단하다. php의 `parse_url` 함수를 사용해서 host, path, query 값을 가져오고 있다. 정규표현식 부분을 보면 path 부분에서만 flag.php를 검사하고 있다.

```php
    <!-- php code -->
    <?php
     $url = $_SERVER['REQUEST_URI'];
     $host = parse_url($url,PHP_URL_HOST);
     $path = parse_url($url,PHP_URL_PATH);
     $query = parse_url($url,PHP_URL_QUERY);
     echo "<div><h1> host: $host <br> path: $path <br> query: $query<br></h1></div>";

     if(preg_match("/flag.php/i", $path)){
        echo "<div><h1>NO....</h1></div>";
     }
     else echo "<div><h1>Cannot access flag.php: $path </h1></div> ";
    ?> 
```

<br>

host로 인식되기 위해선 뒤 경로에 `/` 문자를 2개 입력해주면 된다. 따라서 `//flag.php`를 뒤 경로에 붙여주면 flag가 잘 출력되는 것을 확인할 수 있다. 또한 URL 인코딩으로도 문제를 해결할 수 있다.

![{E9FC4894-15E2-4610-9ECC-6F2806817E29}](/images/2024-12-24-simple-phparse/{E9FC4894-15E2-4610-9ECC-6F2806817E29}.png)