---
layout: single
title: "Frida 기능"
date: 2024-02-09 15:29 +0900
categories: 
    - Android
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

## Frida 명령어

<br>

### frida

- Frida CLI인 Read Evaluate Print Loop(REPL) 인터페이스로, 타겟 앱에 접근할 수 있게 도와준다. Nox 에뮬레이터에서 크롬을 실행 후 `-U` 옵션과 함께 크롬 패키지 명을 입력하여 크롬 프로세스에 접근해서 디버깅이 가능하다. `-U` 옵션은 USB 디바이스를 통해 대상 장치에 연결하는 데 사용된다. 

  ![image-20240209160057837](/images/2024-02-09-android-frida-명령어/image-20240209160057837.png)

<br>

### frida-ps

- 안드로이드에서 실행 중인 프로세스 명령을 출력해준다. `-U` 옵션을 사용해 프로세스 목록을 호출할수 있고, `-a` 옵션을 뒤에 추가하면 실행 중인 앱 목록을 출력해준다. `-i` 옵션까지 추가하면 설치된 앱 목록과 실행 중인 앱 프로세스까지 알려주는 것을 알 수 있다.

![image-20240209153340347](/images/2024-02-09-android-frida-명령어/image-20240209153340347.png)

<br>

### frida-ls-devices

- Frida에 연결된 디바이스를 출력한다.

  ![image-20240209153727803](/images/2024-02-09-android-frida-명령어/image-20240209153727803.png)

<br>

### frida-trace

- Frida가 프로세스의 특정 호출을 동적으로 추적한다. `-i` 옵션을 통해 크롬 앱에서` open` API 함수가 호출될 때마다 로그를 출력하도록 한다.

  ![image-20240209160438357](/images/2024-02-09-android-frida-명령어/image-20240209160438357.png)

<br>

### frida-kill

- 특정 프로세스를 종료할 수 있다. `frida-ps -Ua` 옵션으로 특정 PID를 확인 후 `frida-kill -U 9999` 의 명령처럼 종료가 가능하다.  -U 옵션을 뒤에 넣어주는 것 잊지말자.

  

  ![image-20240209161957648](/images/2024-02-09-android-frida-명령어/image-20240209161957648.png)