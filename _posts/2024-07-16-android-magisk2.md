---
layout: single
title: "Magisk 루팅 및 인증서 설치 세팅"
date: 2024-07-16 23:16 +0900
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

## Magisk로 루팅, SSL인증서, Frida, 디컴파일 세팅

1. [Magisk를 이용한 Android 루팅]을 하고 다음에 다시 AVD를 키기 위해서는 Cold Boot를 해야해서 설정이 초기화됨
2. 기존에 루팅 설정을 했다면 `adb shell`,  `su` 명령을 입력하고 에뮬레이터에서 일괄허용을 눌러주면 루팅 가능
3. 구글 API 기준으로 보안 -> 암호화 및 사용자 인증 정보 -> 인증서 설치 를 통해 crt 확장자 인증서를 사용자로 설치해줌 (openssl을 통한 인증서 설치 과정은 생략), (crt 확장자만 설치 가능)
4. https://github.com/NVISOsecurity/MagiskTrustUserCerts/releases/download/v0.4.1/AlwaysTrustUserCerts.zip 경로를 통해 AlwaysTrustUserCerts 다운로드
5. magisk 모듈에 AlwaysTrustUserCerts.zip 드래그 후 설치 -> 재시작 클릭
6. BurpSuite로 Proxy 설정으로 All Interface 설정해주기 (*:8080)
7. AVD에서 컴퓨터 또는 노트북 IP 프록시 설정해주기 8080 포트
8. AVD 아키텍처에 맞는 Frida 서버 Github에서 다운로드 후 adb push로 /data/local/tmp 폴더로 넣어주기
9. `adb shell`, `su`, `cd /data/local/tmp`, `chmod 777 frida_server~~~'` `./frida_server~~~ &` 명령으로 frida 서버 백그라운드 실행
10. 기존 노트북 환경에 frida와 frida-tools 설치했다면 `frida-ps -Uai` 명령으로 프로세스 잘 잡히는지 확인
11. jadx-gui, apktools, dex2jar, jd-gui 등등 디컴파일 도구 설치
12. 끝
