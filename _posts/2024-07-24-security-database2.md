---
layout: single
title: 데이터베이스 - 시큐리티아카데미 12, 13일차
date: 2024-07-24 20:35 +0900
categories: 
    - Database
#tag: Database
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
published: flase

---

## 데이터베이스

<br>
수업을 진행하면서 Oracle을 기반으로 수업이 진행되었고 https://livesql.oracle.com/ 사이트에서 실습을 진행했다. 아래의 Start Coding을 눌러서 시작할 수 있고, 간단하게 사용법을 설명하자면 강사님께서 테이블을 미리 만들어주셔서 My Scripts 탭을 통해 테이블을 올릴 수 있고 SQL Worksheet를 통해 실습을 진행할 수 있다. 또한 My Session 탭을 통해 이전의 명령들을 볼 수 있고 세션이 끊어지거나 좀 오래되면 테이블이 다 사라지기 때문에 오류가 난다면 테이블을 다시 로드해주자.

![image-20240724203915802](/images/2024-07-24-security-database2/image-20240724203915802.png)

<br>

### 조인

- 두 릴레이션의 공통 속성을 기준으로 속성 값이 같은 투플을 수평으로 결합하는 연산
- 두 릴레이션의 조인에 참여하는 속성은 동일한 도메인으로 구성되어야 함 
- 연산 결과는 공통 속성 값이 동일한 투플만 반환됨 

![image-20240724204348744](/images/2024-07-24-security-database2/image-20240724204348744.png)

<br>

- 세타조인
  - 조인에 참여하는 두 릴레이션의 속성 값을 비교하여 조건을 만족하는 투플만 반환 
- 동등조인
  - 세타조인 중 = 연산자를 사용한 조인만을 말함 
  - 보통 조인 연산이라고 하면 동등조인을 지칭 
- 자연조인
  - 동등조인에서 조인에 참여한 속성이 두 번 나오지 않도록 두 번째 속성을 제거한 결과를 반환
- 외부조인
  - 자연조인 시 조인에 실패한 투플까지 모두 보여주되 값이 없는 대응 속성에는 NULL 값을 채워서 반환
- 세미조인
  - 자연조인을 한 후 두 릴레이션 중 한쪽 릴레이션의 결과만 반환
  - 기호에서 닫힌 쪽 릴레이션의 투플만 반환  

<br>

### SELECT 문의 기본 문법

```sql
SELECT [ALL | DISTINCT] 속성이름
FROM 테이블이름
[WHERE 검색조건]
[GROUP BY 속성이름]
[HAVING 검색조건]
[ORDER BY 속성이름 [ASC | DESC]]
```

<br>

### WHERE

- 비교
- 범위
  - `BETWEEN ~ AND ~`
- 집합
  - `IN, NOT IN`
- 패턴
  - `LIKE`
- NULL
  - `IS NULL, IS NOT NULL`
- 복합조건
  - `AND, OR, NOT`
- 와일드카드 문자의 종류
  - `+, ||, %, [], [^], _`

<br>

### 집계 함수의 종류

- `SUM, AVG COUNT, MAX, MIN`

<br>

### 조인 Oracle

- `WHERE` 조건을 이용해 기본키와 외래키를 이용해 예시로 `WHERE Customer.custid = Orders.custid`로 기본 조인 가능
