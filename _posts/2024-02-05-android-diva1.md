---
layout: single
title: "DIVA 설치 및 실행방법"
date: 2024-02-05 10:00 +0900
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

## DIVA란?

- 안드로이드나 iOS에서 사용하는 애플리케이션에 대해 취약점 진단을 실습할 수 있는 환경을 제공하는 apk
- 2016년 이후에 업데이트 중단됨

<br>

### 실습 취약점 항목

1. 안전하지 않은 로깅
2. 하드코딩 문제
3. 안전하지 않은 데이터 저장
4. 입력 유효성 검사 문제
5. 액세스 제어 문제

<br>

### DIVA 다운로드

[DIVA Github 사이트](https://github.com/0xArab/diva-apk-file)로 접속하여 안드로이드 DIVA apk 파일을 다운로드하면 된다. Nox를 이용하면 간단히 apk 파일을 드래그해서 설치를 할 수 있다.

<br>

설치가 완료되면 다음과 같이 DIVA apk를 이용해 취약점들을 실습할 수 있다.

![image-20240205184426933](/images/2024-02-05-android-diva1/image-20240205184426933.png)

<br>

녹스에 adb를 연결하기 위해 다음과 같은 명령어를 입력하면 된다. 사전으로 adb나 환경변수는 설정해주어야 한다.

![image-20240205193547621](/images/2024-02-05-android-diva1/image-20240205193547621.png)
