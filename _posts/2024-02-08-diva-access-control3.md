---
layout: single
title: "[DIVA] Access Control Issues - Part 3"
date: 2024-02-08 03:58 +0900
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

## 취약한 콘텐츠 프로바이더

DIVA 앱의 11번째 문제인 액세스 제어 Part3는 PIN 4자리를 입력하여 PIN을 생성하고 바꿀 수 있다.

![image-20240208035909902](/images/2024-02-08-diva-access-control3/image-20240208035909902.png)

<br>

핀을 생성하면 아래에 `GO TO PRIVATE NOTES` 버튼이 생기고 핀 번호를 입력하면 노트를 볼 수 있게끔 되어 있다. 이 문제의 취지는 핀을 생성하지 않고 노트를 읽는 것이다.

<img src="/images/2024-02-08-diva-access-control3/image-20240208040018164.png" alt="image-20240208040018164" style="zoom:80%;" />

![image-20240208040031519](/images/2024-02-08-diva-access-control3/image-20240208040031519.png)

![image-20240208040038156](/images/2024-02-08-diva-access-control3/image-20240208040038156.png)

<br>

먼저 AndroidManifest.xml 파일을 살펴보면 provider의 `android:exported`가 true로 설정되어 있음을 알 수 있다.  따라서 외부에서 해당 content를 불러올 수 있다.

```java
<provider android:name="jakhar.aseem.diva.NotesProvider" android:enabled="true" android:exported="true" android:authorities="jakhar.aseem.diva.provider.notesprovider"/>
    <activity android:label="@string/d11" 		          					android:name="jakhar.aseem.diva.AccessControl3Activity"/>
```

<br>

NotesProvider의 소스코드를 살펴보면 provider의 URI의 정보를 확인할 수 있다.

![image-20240208040818343](/images/2024-02-08-diva-access-control3/image-20240208040818343.png)

<br>

이제 adb 쉘에 접속하여 `content query --uri content://jakhar.aseem.diva.provider.notesprovider/notes`명령을 입력하여 노트를 확인할 수 있다. provider 태그는 다른 애플리케이션에 데이터를 제공하는 컴포넌트이다. provider 태그를 사용하여 ContentProvider를 등록하면 해당 프로바이더가 관리하는 데이터에 대한 접근을 허용할 수있다. 이는 `content://` 형식의 URI를 사용하는데 이 URI는 안드로이드에서 데이터에 접근하는 데 사용되는 표준화된 방법이다. 따라서 위의 명령을 사용해서 PIN 없이 노트의 정보를 확인할 수 있는 것이다.  

![image-20240208041116452](/images/2024-02-08-diva-access-control3/image-20240208041116452.png)

<br>

위 말고도 `--projection` 옵션을 사용하여 SQLi를 시도해볼 수 있다. 특수문자 앞에 백슬래쉬(`\`)를 붙여 시도해볼 수 있다. 또한 콘텐츠 프로바이더 취약점을 바로 확인하기 위해서 jadx 도구에서 텍스트 검색에서 `content://`를 찾아볼 수도 있다.

<br>

### 대응 방안

1. 모든 액티비티에 권한 기능 추가
2. exported 속성을 false로 설정
3. protectionLevel을 signature로 설정
4. 루팅 또는 애뮬레이터 환경 체크 기능 추가