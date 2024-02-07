---
layout: single
title: "[DIVA] Hardcoding Issues - Part 2"
date: 2024-02-08 04:29 +0900
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

## 하드코딩된 중요정보 노출

DIVA의 12번째의 하드코딩된 중요정보 노출 취약점을 실습해 볼 것이다.

<img src="/images/2024-02-08-diva-hardcoding-issues2/image-20240208043022673.png" alt="image-20240208043022673" style="zoom:80%;" />

<br>

이 액티비티에서 문자를 입력하고 ACCESS 버튼을 누르면 Access denied라는 토스트 알림이 출력되며 오류가 발생한다. 토스트(Toast)란 안드로이드에서 사용하는 팝업 메시지 알림 창을 말한다. 

![image-20240208043254006](/images/2024-02-08-diva-hardcoding-issues2/image-20240208043254006.png)

<br>

jadx 도구를 이용해 소스 코드를 먼저 확인해보면 아래와 같이 `this.djni.access(hckey.getText().toString()) != 0`의 조건을 이용해 비교하고 있다.

![image-20240208043537785](/images/2024-02-08-diva-hardcoding-issues2/image-20240208043537785.png)

<br>

djni.access를 더블클릭하여 어떤 소스 코드인지 확인하면 `divajni`라는 라이브러리에서 값을 반환한다.

![image-20240208043743992](/images/2024-02-08-diva-hardcoding-issues2/image-20240208043743992.png)

<br>
라이브러리 파일을 분석하기 위해선 jadx 도구를 사용할 수 없고, 기드라를 통해 분석할 수 있다. so 파일은 aremabi 폴더 아래에 있다.

![image-20240208044109908](/images/2024-02-08-diva-hardcoding-issues2/image-20240208044109908.png)

<br>
아래와 같이 Ghidra를 열어 프로젝트를 생성해준다.

![image-20240208045209447](/images/2024-02-08-diva-hardcoding-issues2/image-20240208045209447.png)

<br>
그 전에 먼저 so 파일을 임포트하기 위해 디컴파일을 해주어야 한다. apktool을 설치하고 환경변수까지 설정하였다면 해당 apk 경로에서 `apktool d "파일경로"` 를 입력해 디컴파일을 할 수 있다.

![image-20240208050255604](/images/2024-02-08-diva-hardcoding-issues2/image-20240208050255604.png)

<br>

그럼 해당 디컴파일 폴더의 lib -> armeabi 폴더에 so 파일을 볼 수 있다.

![image-20240208050406722](/images/2024-02-08-diva-hardcoding-issues2/image-20240208050406722.png)

<br>
so 파일을 import 하기 위해 클릭한다.

![image-20240208050500509](/images/2024-02-08-diva-hardcoding-issues2/image-20240208050500509.png)

<br>
아무것도 건들이지 않고 ok -> ok 버튼 클릭한다.

![image-20240208050655904](/images/2024-02-08-diva-hardcoding-issues2/image-20240208050655904.png)

<br>

추가된 라이브러리 파일을 더블클릭하고 yes 버튼을 누른다.

![image-20240208050732812](/images/2024-02-08-diva-hardcoding-issues2/image-20240208050732812.png)

<br>
아무 것도 건들이지 않고 Analyze를 누르고 기다리면 아래와 같은 창이 뜬다.

![image-20240208050849226](/images/2024-02-08-diva-hardcoding-issues2/image-20240208050849226.png)

<br>
그럼 Source Tree에서 아까 access 문구가 있었으므로 filter에 access를 검색하고 Divjani.access를 눌러 오른쪽 디컴파일 소스를 보면 `uVar1` 변수에 `olsdfgad;lh`와 비교하고 있는 것을 알 수 있다.

![image-20240208051159762](/images/2024-02-08-diva-hardcoding-issues2/image-20240208051159762.png)

<br>DIVA 앱에서 아래의 문자열을 입력하면 된다.

![image-20240208051428873](/images/2024-02-08-diva-hardcoding-issues2/image-20240208051428873.png)