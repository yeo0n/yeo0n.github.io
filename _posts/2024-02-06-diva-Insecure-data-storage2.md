---
layout: single
title: "[DIVA] Insecure Data Storage - Part 2"
date: 2024-02-06 02:21 +0900
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

## 로컬 저장소 내 평문 저장된 중요 정보 확인 - II

<br>
### Insecure Data Storage 취약점이란

- 안드로이드 앱은 로컬 저장소에 파일을 저장할 수 있다.
- 환경 설정 정보, 외부 API 연동 및 인증을 위한 토큰 값, 연동 서버 정보 등이 로컬 스토리지에 저장된다.
- 루팅된 환경에서는 애플리케이션이 다른 애플리케이션의 디렉토리에 접근할 수 있으므로 데이터를 탈취할 수 있다.
- 중요 데이터가 평문 형태로 제3자에게 유출될 경우 2차 피해로 이어질 수 있다.

<br>
DIVA 앱에서 아래와 같이 Insecure Data Storage - Part 2 액티비티에 들어가준 후, `name`과 `password`를 각각 test2, password2로 설정하고 save 버튼을 눌러주었다.

![image-20240206023222122](/images/2024-02-05-diva-Insecure-data-storage2/image-20240206023222122.png)

<br>

`nox_adb shell` 명령을 통해 adb 연결을 해준 후 전과 같이 로컬 저장소 내 데이터들은 `/data/data/[package_name]` 경로에 저장될 확률이 높다. 따라서 아래 경로에서 최근 변경된 데이터를 확인해주기 위해 `ls -alR` 명령을 입력해주면 시간대별로 변경한 하위 디렉토리의 파일과 폴더까지 출력할 수 있다. 보면 `databases` 폴더에서 `ida2`와 `ida2-journal`이 추가된 것을 확인할 수 있다.

![image-20240206023637857](/images/2024-02-05-diva-Insecure-data-storage2/image-20240206023637857.png)

<br>
위의 `journal` 파일은 SQLite에서 write 이전으로 되돌아가고자 할 때 사용하는 롤백파일이므로 무시하면 된다. `ids2` 파일을 확인하기 위해 이 파일을 현재 로컬 컴퓨터로 `pull`명령어를 이용해 옮겨줄 수 있다.  아래와 같은 명령을 통해 `ids2`를 옮겨주자.

```bash
exit
nox_adb pull /data/data/jakhar.assem.diva/databases/ids2
```

![image-20240206031327770](/images/2024-02-05-diva-Insecure-data-storage2/image-20240206031327770.png)

<br>

만약에 위 과정이 수행이 안된다면 Nox 에뮬레이터를 킨 후에 터미널에 `adb devices -l` 명령이나 아마도 Nox 에뮬레이터를 켰으면 `nox_adb devices -l` 명령어를 입력하면 127.0.0.1:62001 같은 주소와 포트가 뜨는데, 이를 adb와 연결해준다. `adb connect 127.0.0.1:62001`과 같은 명령으로 adb를 연결해주고 위의 pull 명령으로 ids2를 현재 로컬 컴퓨터로 복사해오면 된다. 

<br>

SQLite를 실행한 후 데이터베이스 열기로 ids2를 선택하면 다음과 같이 데이터베이스 구조 옆에 데이터 보기 탭이 존재한다. 이를 클릭한다.

![image-20240206031818510](/images/2024-02-05-diva-Insecure-data-storage2/image-20240206031818510.png)

<br>
데이터 보기에서 myuser 테이블의 값을 확인해보면 아래와 같이 user와 password 컬럼에 3rd party 계정 정보가 그대로 평문으로 노출되어 있는 것을 확인할 수 있다.

![image-20240206031945850](/images/2024-02-05-diva-Insecure-data-storage2/image-20240206031945850.png)

