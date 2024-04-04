---
layout: single
title: "함수 호출 규약"
date: 2024-04-04 21:55 +0900
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



## 함수 호출 규약

- 함수 호출 규약은 함수의 호출 및 반환에 대한 약속
- 함수를 호출할 때는 반환된 이후를 위해 호출자(Caller)의 상태(Stack frame) 및 반환 주소(Return address)를 저장해야 함 
- 호출자는 피호출자(Callee)가 요구하는 인자를 전달해줘야 하며, 피호출자의 실행이 종료될 때에는 반환 값을 전달받아야 함
- 함수 호출 규약을 적용하는 것은 일반적으로 컴파일러의 몫이지만 컴파일러의 도움 없이 어셈블리 코드를 작성하거나 어셈블리로 작성된 코드를 읽고자 한다면 함수 호출 규약을 알아야 함

<br>

### 함수 호출 규약 종류

- 컴파일러는 지원하는 호출 규약 중, CPU 아키텍처에 적합한 것을 선택함
  - x86은 레지스터 수가 적으므로 스택을 사용, x86_64는 레지스터 수가 많으므로 레지스터 사용 후 부족하면 스택 사용
- CPU의 아키텍처가 같아도 컴파일러가 다르면 적용하는 호출 규약이 다름 
  - C언어를 컴파일할 때, 윈도우는 MSVC, 리눅스는 gcc를 사용하는데 다른 호출 규약을 적용함 
  - x86-64 아키텍처에서 MSVC는 MS x64 호출 규약을 적용하지만, gcc는 SYSTEM V 호출 규약을 적용함 

- x86 호출 규약
  - cdecl
  - stdcall
  - fastcall
  - thiscall
- x86-64 호출 규약
  - SYSTEM V AMD64 ABI의 Calling Convention
  - MS ABI의 Calling Convention

<br>

### x86-64 호출 규약: SYSV

- SYSV ABI는 ELF 포맷, 링킹 방법, 함수 호출 규약 등의 내용을 담고 있음 

- file 명령어를 이용해 바이너리 정보를 살펴보면 SYSV 문자열 확인 가능

  ![image-20240404221825342](/images/2024-04-04-calling convention/image-20240404221825342.png)

- **SYSV 함수 호출 규약의 특징**

  - 6개의 인자 rdi,rsi,rdx,rcx,r8,r9에 순서대로 저장하여 전달하며, 더 많은 인자를 사용할 때는 스택을 사용함
  - Caller에서 인자 전달에 사용된 스택을 정리함 
  - 함수의 반환 값은 RAX로 전달함

<br>

### x86 호출 규약: cdecl 

- x86아키텍처는 레지스터의 수가 적으므로 스택을 통해 인자를 전달함 
- 인자를 전달하기 위해 스택을 호출자가 정리하는 특징이 있음 
  - 스택을 통해 인자를 전달할 때 마지막 인자부터 첫 번째 인자까지 스택에 거꾸로 push함 
- 예를 들어 만약 hello이라는 함수에 인자가 3개가 있다고 하였을 때,  3번째 인자부터 거꾸로 push하고, 이 인자가 3개이므로 hello를 call하여 함수를 실행한 후 x86 아키텍처는 스택의 데이터 단위가 4바이트이기 때문에, 총 3개이므로 0xc 만큼 esp에 add 명령을 통해 더해주어야 한다. 



