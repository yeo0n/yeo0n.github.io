---
layout: single
title: "Return to Library"
date: 2024-04-06 15:27 +0900
categories: 
    - SYSTEM-study
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



## Return to Library 

- NX 보호 기법은 각 세그먼트에 불필요한 실행 권한을 제거함으로써 공격자가 임의 버퍼에 주입한 코들르 실행하기 어렵게 함.
  - 이는 코드영역과 데이터영역은 읽기와 실행 권한, 나머지 영역은 읽기와 쓰기 권한이 부여
- NX를 우회하는 기법이 Return to Library(RTL) 기법임
- 유사한 기법으로 Return to PLT가 있는데 이 공격 기법도 라이브러리 코드를 사용하는 것이 핵심이므로 RLT의 하위 분류임

<br/>

- PLT에 어떤 라이브러리 함수가 등록되어 있다면, 그 함수의 PLT 엔트리를 실행함으로써 함수를 실행할 수 있음 

- ASLR이 걸려 있어도 PIE가 적용되어 있지 않다면 PLT의 주소는 고정됨
- 위처럼 PIE가 적용되어 있지 않고, 라이브러리 베이스 주소를 몰라도 위 방법으로 라이브러리 함수를 실행할 수 있고, 이 방법을 Return to PLT라고 부름 

<br/>

### 익스플로잇 

- system이나 execve 같은 함수를 실행해야 하므로, 이 함수가 코드에서 사용되어 PLT에 있어야 함
- 만약 /bin/sh 의 주소를 알고, system PLT 주소를 알면 system 함수를 호출할 수 있음
- PLT 주소는 pwndbg에서 `plt` 나` info func @plt`를 통해 알 수 있음 
- rdi가 x86-64 아키텍처의 첫 번째 인자로 들어가므로, /bin/bash를 rdi 인자에 넣어야 함
- system 함수는  movaps 명령으로 인해 호출 전 스택을 0x10으로 맞춰주어야 함. 따라서 필요에 따라 의미 없는`ret` 가젯을 더 추가해주어야 함
  - 올바르게 익스 코드를 작성하였는데 segmentation fault 에러가 발생한다면 위 방법을 검토
- 리턴 가젯은 ret 명령어로 끝나는 어셈블리 코드 조각으로 `ROPgadget --binary ./test --re "pop rdi"` 같은 명령으로 정규표현식을 사용해 pop rdi를 포함하는 가젯만 뽑아낼 수 있음 



