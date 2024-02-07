---
layout: single
title: "[DIVA] Insecure Data Storage - Part 4"
date: 2024-02-07 23:30 +0900
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

## Insecure Data Storage

DIVA앱의 Part4의 Insecure Data Storage를 실습하기 위해선 먼저 아래와 같이 저장 권한을 활성화 하여야 한다.

- 설정 -> 애플리케이션 -> DIVA -> 권한 -> 저장 활성화

![image-20240208005037124](/images/2024-02-07-diva-Insecure-data-storage4/image-20240208005037124.png)

<br>
DIVA앱에서 아래와 같이 3rd party 계정의 name과 password를 `test4`, `password4`로 설정했다. 여기서 저장권한을 활성화하지 않으면 File error occurred라는 오류 문구가 출력된다.

![image-20240208005305265](/images/2024-02-07-diva-Insecure-data-storage4/image-20240208005305265.png)

<br>
`ls -alR` 명령으로 adb 쉘에서 `/data/data/jakhar.aseem.diva` 경로를 확인하면 변경되거나 추가된 파일이 보이지 않는다.

![image-20240208005459420](/images/2024-02-07-diva-Insecure-data-storage4/image-20240208005459420.png)

<br>다음으로 외부저장소인 `/sdcard` 경로에서 파일을 확인하면 아래와 같이 `.uinfo.txt` 파일이 생성된 것을 확인할 수 있다. 참고로 `/data/data/<패키지>` 경로는 앱의 내부 저장소를 의미하고 앱이 생성한 데이터와 설정 파일 등이 저장된다. 내부저장소는 앱의 프라이빗한 공간으로, 다른 앱이나 사용자가 직접 액세스할 수 없지만 루팅하면 가능하다. 또한 `/sdcard` 경로는 외부 저장소를 의미하고 이 경로는 기기의 외부 저장장치를 가맃키며, 사용자가 파일을 저장하고 공유할 수 있는 공용 공간이다. 외부 저장소에 저장된 파일은 사용자 또는 다른 앱이 액세스할 수 있다.

![image-20240208005737848](/images/2024-02-07-diva-Insecure-data-storage4/image-20240208005737848.png)

<br>
`.uinfo.txt`파일을 확인하면 DIVA앱에서 설정한 계정이 그대로 평문으로 노출되고 있는 것을 알 수 있다. 

![image-20240208010134692](/images/2024-02-07-diva-Insecure-data-storage4/image-20240208010134692.png)

<br>

jadx 도구로 Insecure Data Storage4의 소스 코드를 확인해보면 전과 동일하게 `.getText()` 함수와 `.toString()` 함수를 사용해 평문으로 uinfo 파일을 생성하는 것을 알 수 있다. 

![image-20240208010239356](/images/2024-02-07-diva-Insecure-data-storage4/image-20240208010239356.png)