---
layout: single
title: "[mobilehacking.kr] Incident Response Scenario 1 - Tracking the Infiltration Path writeup"
date: 2026-01-02 20:50 +0900
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

## [Mobilehacking.kr] Incident Response Scenario 1 - Tracking the Infiltration Path writeup

### 😜 문제 설명

- Incident Response Background: You are a forensic analyst tasked with analyzing an Android phone where a card information leak incident has occurred. The user reported that their card information was leaked after recently installing a mobile payment-related app. Analyze the Android device image seized by law enforcement to uncover the full details of the incident. Tracking the Infiltration Path Description: The user stated that they installed a card management app a few days ago. You need to identify how the app was installed and what information was exposed during the installation process. **Your Mission:** 1. Find the path through which the user was directed to the app installation page (ex: http://10.1.2.3:1234) 2. Identify the format of sensitive information exposed in external storage (ex: card number) 3. Identify the app that actually collected the card information (ex: com.android.card) Combining these three clues will give you the first flag. **Flag Format:** `flag{clue1_clue2_clue3}`

<br>

### ✏️ 풀이

문제 설명을 보면 Mission에 대해서 Flag Format에 맞춰 작성해야 한다.

1. 앱 설치 경로 찾기
2. 외부 저장소에 노출된 민감정보
3. 패키지명

<br>

안드로이드 이미지 파일을 다운받고 vscode로 열어서 `mnt/user/0/Download` 폴더를 열어보니 설치된 앱과 카드번호가 txt 파일로 노출되어 있는 것을 확인했다. 여기서 바로 clue2 값은 `211250980212525` 인 것을 확인할 수 있다.

![image-20260102233428087](/images/2026-01-02-mobilehacking-mobilebabyanalysis2/image-20260102233428087.png)

<br>

해당 카드번호로 code에서 검색해보니 `com.qr.cardgen` 패키지명이 나온 것을 알 수 있다.

![image-20260102233654199](/images/2026-01-02-mobilehacking-mobilebabyanalysis2/image-20260102233654199.png)

<br>

이제 문제는 해당 패키지를 설치하게된 경로를 알아내야 한다. 이는 `/data/data/com.android/provider/downloads/databases/` 경로의 downloads.db 파일을 분석해보면 downloads 테이블에 해당 앱을 설치한 uri가 포함되어 나오는 것을 알 수 있다. 따라서 clue1 값은 `http://172.30.1.61:8088` 이 된다.

![image-20260102233911807](/images/2026-01-02-mobilehacking-mobilebabyanalysis2/image-20260102233911807.png)

<br>

이거 때문에 삽질을 좀 하게 됐는데, 카드 정보를 수집한 패키지가 com.qr.cardgen 이라고 착각하고 있었다. 실제 저장된 카드 정보는 /data/data/ 폴더에 com.card.safe 패키지에서 `card_info` 테이블에 저장되고 있었기 때문에 clue3 값은 `com.card.safe` 인 것을 알 수 있다.

![image-20260102234251404](/images/2026-01-02-mobilehacking-mobilebabyanalysis2/image-20260102234251404.png)

<br>

위 clue1,2,3 값을 flag 포멧에 맞춰 연결해주면 해결할 수 있다.

![image-20260102234817321](/images/2026-01-02-mobilehacking-mobilebabyanalysis2/image-20260102234817321.png)
