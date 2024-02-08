---
layout: single
title: "안드로이드 Frida 세팅"
date: 2024-02-08 22:07 +0900
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

## Frida 세팅

먼저 PC에서 python을 설치한 후 Frida를 설치한다. 

`pip install frida-tools`

![image-20240208220907602](/images/2024-02-08-android-frida-setting/image-20240208220907602.png)

<br>

Frida를 설치하였으면 설치한 Frida 버전과 단말기의 환경을 먼저 확인해야 한다. Frida 버전 확인은 `frida --version`을 통해 확인할 수 있다.

![image-20240208221019725](/images/2024-02-08-android-frida-setting/image-20240208221019725.png)

<br>

이제 단말기의 아키텍처를 확인해야 하는데, adb 쉘에서 `getprop ro.product.cpu.abi` 명령으로 아키텍처를 확인할 수 있다.

![image-20240208221100566](/images/2024-02-08-android-frida-setting/image-20240208221100566.png)

<br>

아키텍처를 확인 후, [Frida Github](https://github.com/frida/frida/releases)에서 자신이 설치한 버전과 아키텍처에 맞는 서버를 다운로드한다.

![image-20240208232210994](/images/2024-02-08-android-frida-setting/image-20240208232210994.png)

<br>
압축을 해제하고 adb를 이용해 다운받은 frida-server를 압축 해제 후, push해주고 실행한다. 

![image-20240208234627531](/images/2024-02-08-android-frida-setting/image-20240208234627531.png)

![image-20240208234607080](/images/2024-02-08-android-frida-setting/image-20240208234607080.png)

<br>

Frida 서버가 정상 동작 하는 것을 확인하면 PC 명령 창에서 `frida-ps -U`를 입력하여 아래와 같은 결과가 나온다면 구축 성공이다.

![image-20240209003140280](/images/2024-02-08-android-frida-setting/image-20240209003140280.png)





















