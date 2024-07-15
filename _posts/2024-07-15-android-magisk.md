---
layout: single
title: "M3에서 Magisk를 이용한 Android 루팅"
date: 2024-07-15 20:36 +0900
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

## Magisk를 이용한 Android 루팅

<br>

1.  [https://github.com/newbit1/rootAVD](https://github.com/newbit1/rootAVD)  <- 해당 사이트에서 git clone
2. Android Studio에서 AVD 생성 (플레이스토어 없는걸로 생성함)
3. `adb devices` 명령어로 avd 연결
4. 터미널을 이용해서 rootAVD 폴더로 이동
5. `./rootAVD.sh ListAllAVDs`  명령어 입력 후 바로 아래 나오는 avd의 api에 맞는 ramdisk.img 경로 복사
6. `./rootAVD.sh system-images/android-30/google_apis/arm64-v8a/ramdisk.img` 입력 후 AVD가 자동으로 꺼질거임
7. Android Studio를 이용해서 Cold Boot 해야 함 
8. `adb shell`, `su` 명령을 입력하고 AVD에서 권한 허용
9. 끝 

![image-20240715204151528](/images/2024-07-15-android-magisk/image-20240715204151528.png)

<img src="/images/2024-07-15-android-magisk/image-20240715204207956.png" alt="image-20240715204207956" style="zoom:50%;" />