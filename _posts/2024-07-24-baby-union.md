---
layout: single
title: "[Dreamhack] baby-union write-up"
date: 2024-07-24 21:26 +0900
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

## [Dreamhack] baby-union

<br>

### 문제 설명

로그인 시 계정의 정보가 출력되는 웹 서비스입니다.
SQL INJECTION 취약점을 통해 플래그를 획득하세요. 문제에서 주어진 `init.sql` 파일의 테이블명과 컬럼명은 실제 이름과 다릅니다.

플래그 형식은 `DH{...}` 입니다.

<br>

### 풀이

접속하면 아래와 같이 SQL 쿼리가 보이고 uid와 upw 값을 입력하는 폼이 있는 것을 볼 수 있다.

![image-20240724213118684](/images/2024-07-24-baby-union/image-20240724213118684.png)

<br>
init.sql 파일을 먼저 확인해보면 users 테이블에는 idx, uid, upw, descr의 컬럼이 있고, INSERT 문을 통해 각각 admin, guest, banana의 계정들을 upw, descr값들과 함께 넣은 것을 볼 수 있다.

![image-20240724213234053](/images/2024-07-24-baby-union/image-20240724213234053.png)

<br>
소스 코드를 살펴보면 POST 메소드로 uid와 upw를 넣어주면 아래 보이는 SQL 쿼리문에 각각 값을 삽입하는 것을 알 수 있다.

![image-20240724213628356](/images/2024-07-24-baby-union/image-20240724213628356.png)

<br>
먼저 guest, melon 값을 넣어보면 아래와 같이 Hello guest가 나오는 것을 볼 수 있다. 또한 admin과 apple을 입력해봐도 특별히 별다른게 없다.

![image-20240724213727869](/images/2024-07-24-baby-union/image-20240724213727869.png)

![image-20240724213805642](/images/2024-07-24-baby-union/image-20240724213805642.png)

<br>
해당 쿼리는 아무 값도 필터링 하지 않기 때문에 먼저 UNION을 통해 select version()과 함께 컬럼이 4개인 것을 알 수 있고, MariaDB를 사용하는 것도 알 수 있다. 

```
admin' UNION SELECT version(), 1, 2, 3# 
```

![image-20240724215753642](/images/2024-07-24-baby-union/image-20240724215753642.png)

<br>

그럼 MariaDB의 기본 스키마는 `information_schema` 이기 때문에 테이블 정보를 첫 번째에 알아내기 위해 SELECT table_name으로 `information_schema.tables`안에 있는 테이블들을 나열할 수 있다. 그럼 다음과 같이 users와 onlyflag의 테이블을 확인할 수 있다.

```
admin' UNION SELECT table_name, 1, 2, 3 from information_schema.tables #
```

![image-20240724220139383](/images/2024-07-24-baby-union/image-20240724220139383.png)

<br>
테이블이 딱봐도 수상하게  onlyflag로 되어 있으므로 이제 이 onlyflag의 컬럼 값을 알기 위해 table_name을 column_name으로 바꿔주고 WHERE 절을 추가해 table_name이 onlyflag인 조건을 추가해주면 된다. 그럼 아래와 같이 `idx, sname, svalue, sflag, sclose` 컬럼을 확인할 수 있다.

```
admin' UNION SELECT column_name, 1, 2, 3 from information_schema.columns WHERE table_name = 'onlyflag' #
```

![image-20240724220545727](/images/2024-07-24-baby-union/image-20240724220545727.png)

<br>

그럼 이제 idx는 기본적으로 증가하는 컬럼이므로 나머지 것들만 select해서 onlyflag 테이블을 조회해보면 아래와 같이 나온다. 이어서 붙여보니 해당 답이 아니라고 나온다.

```
admin' UNION SELECT sname, svalue, sflag, sclose from onlyflag #
```

![image-20240724220844062](/images/2024-07-24-baby-union/image-20240724220844062.png)

<br>
다시 잘 살펴보면 컬럼을 보여주는 부분이 3개밖에 존재하지 않고, 보여지는 부분을 테스트 해보니 3번째 부분이 보이지 않는다. 따라서 flag is 부분인 sname을 없애주고 아래와 같이 null을 넣어주면 flag 값을 확인할 수 있다.

```
admin' UNION SELECT svalue, sflag, null, sclose from onlyflag #
```

![image-20240724221225281](/images/2024-07-24-baby-union/image-20240724221225281.png)