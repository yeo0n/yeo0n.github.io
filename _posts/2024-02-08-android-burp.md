---
layout: single
title: "안드로이드 Burp Suite 프록시 세팅"
date: 2024-02-08 20:23 +0900
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

## 프록시 툴 세팅

1. 프록시 툴 IP 및 포트 설정
2. 모바일 단말기 Wi-Fi 및 프록시 설정
3. 프록시 툴 인증서 등록

<br>

### 프록시 툴 IP 및 포트 설정

Burp Suite 툴을 사용할 것이고, 프록시 IP와 포트를 새로 다시 설정해주어야 한다. 먼저 기본적으로 127.0.0.1:8080으로 설정되어 있을텐데, 모바일과 같은 Wi-Fi이거나 같은 네트워크 환경으로 셋팅해준 후 프록시의 IP와 포트를 컴퓨터의 IPv4의 IP와 임의의 포트를 설정해준다.

![image-20240208203857666](/images/2024-02-08-android-burp/image-20240208203857666.png)

<br>
Nox 에뮬레이터에서는 Wi-Fi 탭에서 와이파이를 길게 누르고 있으면 Wi-Fi 설정 탭이 보인다. 선택한 후 고급에서 수동으로 변경하고, 프록시 호스트 이름과 포트를 burp suite에서 설정한 것과 동일하게 설정해준다.

<img src="/images/2024-02-08-android-burp/image-20240208204156969.png" alt="image-20240208204156969" style="zoom:80%;" />

<br>

`cacert.der`이라는 der 확장자의 인증서를 burp suite에서 다운로드 받는다. 다운로드 받았다면 openssl을 설치하고 환경변수까지 설정해주자.

![image-20240208204820590](/images/2024-02-08-android-burp/image-20240208204820590.png)

<br>

이제 openssl을 통해서 DER을 PEM으로 변경한다.

`openssl x509 -inform DER -in cacert.der -out cacert.pem`

![image-20240208210450258](/images/2024-02-08-android-burp/image-20240208210450258.png)

<br>

인증서 해시값을 추출한다. 아래 보이는 `9a5ba575`를 복사해준다.

`openssl x509 -inform PEM -subject_hash_old -in cacert.pem`

![image-20240208210611013](/images/2024-02-08-android-burp/image-20240208210611013.png)

<br>

PEM 파일 명을 `해시값.0`으로 변경한다.

![image-20240208210754591](/images/2024-02-08-android-burp/image-20240208210754591.png)

<br>

adb를 이용해 `/system/etc/security/cacerts/`로 옮긴다.  이 과정에서 오류가 발생할 경우 아래와 같은 명령을 시도해볼 수 있다.

![image-20240208211120575](/images/2024-02-08-android-burp/image-20240208211120575.png)

```text
mount -o re, remount /system
mv /data/local/tmp/9a5ba575.0 /system/etc/security/cacerts/
mount -o ro, remount /system
```

<br>

Nox 에뮬레이터에서는 설정 -> 보안 -> 신뢰할 수 있는 자격증명 탭에서 PortSwigger의 인증서가 설치되어 있는 것을 확인할 수 있다.

<img src="/images/2024-02-08-android-burp/image-20240208212441668.png" alt="image-20240208212441668" style="zoom:80%;" />

<br>

이제 Burp Suite에서 Intercept를 키고 패킷을 잡으면 잘 잡히는 것을 볼 수 있다. 

![image-20240208220349215](/images/2024-02-08-android-burp/image-20240208220349215.png)