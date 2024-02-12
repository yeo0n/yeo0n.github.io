---
layout: single
title: "안드로이드 APK 파일 인증서 등록"
date: 2024-02-09 03:03 +0900
categories: 
    - Android
tag: 
#typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---

## APK 파일 인증서 등록이란?

- Code Sign 또는 앱 서명이라고도 부르며 APK 파일에 인증서를 등록하는 과정을 말한다.
- 인증서를 등록하지 않으면 리패키징한 APK 파일이 정상적으로 설치 또는 실행이 되지 않는다.

<br>

먼저 인증서를 등록하기에 앞서 APK 파일을 디컴파일한 후, 수정하고 다시 리패키징을 해주어야 하는데 apktool 도구를 이용하면 된다.  

apktool 도구가 환경변수 설정이 되어있다면 apk의 경로에서 `apktool d "<apk파일명>" ` 을 입력하면 경로에 디컴파일 폴더가 생성된다.  

리패키징은 `apktool b "<디컴파일된폴더명>"` 을 입력하면 폴더안에 dist라는 폴더가 생성되고 apk 파일을 볼 수 있다.

<br>

### jarsigner

JAVA를 설치하였다면 jarsigner.exe를 이용해 인증서를 등록할 수 있다. 관리자 권한으로 cmd를 실행하고, bin 폴더로 이동한다.

![image-20240209030952668](/images/2024-02-09-android-sign/image-20240209030952668.png)



<br>
bin 폴더 내에 존재하는 keytool을 이용해 keystore를 생성한다. 

`keytool -genkey -v -keystore <임의파일명.keystore> -alias <임의별칭> -keyalg RSA -keysize 2048` 명령어를 입력하여 keystore를 생성할 수 있다.  다음으로 패스워드나 정보들을 임의로 입력해주면 되겠다.

<br>

`jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore <생성한keystore파일경로> <리패키징할APK경로> -signedjar <인증서등록한APK저장경로> <지정한별칭>` 명령을 입력하고 패스워드를 입력하면 인증서 등록이 완료된다.

<br>

### 인증서 등록한 APK 파일에서 오류 발생 시

- 어플설치 실패나, 패키지가 잘못되어 앱이 설치되지 않았습니다 등의 오류 발생시 AndroidManifest.xml 파일을 수정한다.
- `android:extractNativeLibs="false"`를 true로 변경한다음 다시 리패키징, 인증서 등록 후 설치한다.