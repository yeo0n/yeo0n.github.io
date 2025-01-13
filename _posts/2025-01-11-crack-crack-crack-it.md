---
layout: single
title: "[Dreamhack] crack-crack-crack-it write-up"
date: 2025-01-11 19:30 +0900
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

## [Dreamhack] crack crack crack it

### 😊 문제 설명

```text
.htaccess crack!

can you local bruteforce attack?
```

<br>

### ✏️ 풀이

문제 사이트에 들어가보면 패스워드가 `G4HeulB`로 시작하고 숫자와 영어 알파벳으로 이루어졌다고 한다. 그리고 해당 문제에서는 따로 문제 파일은 존재하지 않는다.

![{FE62436F-030C-4597-BA90-931D54BCFB86}](/images/2025-01-11-crack-crack-crack-it/{FE62436F-030C-4597-BA90-931D54BCFB86}.png)

<br>

htpasswd 값을 확인해보면 아래와 같은데 etc/shadow 파일과 같이 되어 있다. blueh4g는 유저 이름, $1은 md5, $Xow4VCAg는 솔트, $.zO6fNYB6zEJSzd.PhNri0는 해시된 패스워드이다.

```text
blueh4g:$1$XoW4VCAg$.zO6fNYB6zEJSzd.PhNri0
```



<br>

kali에 있는 john the ripper를 사용해서 아래와 같이 --1=[0-9a-z]은 숫자와 영어 알파벳 소문자를 의미하고, ?1을 통해 앞에 있는 숫자와 영어 알파벳을 사용하면서 `G4HeulB`로 시작한다고 해주었다. 다음으로 --min-length에는 최소 길이 8과 다음은 htpasswd파일 경로를 입력 해주면 된다.

![{76938082-7A58-42E7-8B71-2289F272C652}](/images/2025-01-11-crack-crack-crack-it/{76938082-7A58-42E7-8B71-2289F272C652}.png)

<br>

아래 명령어로 입력해주면 크랙된 패스워드 결과를 확인할 수 있다. 비밀번호를 아까 그 전에 폼에 입력하면 flag가 나온다.

```text
john --show ./htpasswd
```

![{F957608E-E1DA-48CF-991B-59C7FDE224A3}](/images/2025-01-11-crack-crack-crack-it/{F957608E-E1DA-48CF-991B-59C7FDE224A3}.png)

![{516A268D-CCFA-4E76-8BF9-D48C73098501}](/images/2025-01-11-crack-crack-crack-it/{516A268D-CCFA-4E76-8BF9-D48C73098501}.png)