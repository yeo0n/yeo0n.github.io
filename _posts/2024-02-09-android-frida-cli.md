---
layout: single
title: "Frida js 파일 사용법"
date: 2024-02-09 19:49 +0900
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

## Frida js 파일 사용법

- Frida를 이용해 js 파일을 사용하기 위해서는 사전 작업이 필요하다.
- 방법은 여러가지가 있겠지만 대표적인 2가지는 다음과 같다.
  - 앱 프로세스 실행 후 자바스크립트 삽입
  - 앱 프로세스 실행 전 자바스크립트 삽입

<br>

### 앱 프로세스 실행 후 자바스크립트 삽입

- Frida를 이용해 후킹하기 위해선, 단말기나 에뮬레이터에서 특정 앱을 먼저 실행시키고 패키지 이름을 확인한 후, `-U` 옵션을 이용하면 된다.

  ![image-20240209195508113](/images/2024-02-09-android-frida-cli/image-20240209195508113.png)

<br>

### 앱 프로세스 실행 전 자바스크립트 삽입

- 앱 프로세스 실행 전에 자바스크립트를 삽입하려면 프로세스를 spawn하여 자동으로 실행되게 한 후 접근한다. spawn은 `-f`옵션을 사용하면 된다.

  ![image-20240209195646634](/images/2024-02-09-android-frida-cli/image-20240209195646634.png)

- `--no-pause` 옵션이 있다고 보았는데 이는 frida 15.2.2 버전 이상부터 더 이상 기본적으로 일시 중지 되지 않으므로 삭제된 것 같다.
- `--pause` 옵션을 통해 대상 프로세스에 연결될 때, 프로세스의 실행을 일시 중지 시킬 수 있다. 이 명령은 Frida가 프로세스에 영향을 주기 전에 프로세스를 안전하게 조작하고 분석할 수 있도록 해주며, `%resume` 명령을 통해서 프로세스를 다시 재개할 수 있다.

<br>

### jadx frida 스니펫 이용하기

- jadx 도구는 아래와 같이 firda 스니펫 기능을 제공해준다. 따라서 이를 이용해 사용자가 조금 더 쉽게 firda 코드를 작성할 수 있다.

![image-20240209212700041](/images/2024-02-09-android-frida-cli/image-20240209212700041.png)

```javascript
let Hardcode2Activity = Java.use("jakhar.aseem.diva.Hardcode2Activity");
Hardcode2Activity["access"].implementation = function (view) {
    console.log(`Hardcode2Activity.access is called: view=${view}`);
    this["access"](view);
};
```



<br>

### 로드된 모든 클래스를 열거하여 일치 항목 출력

- 아래 코드를 이용해서 모든 클래스를 열거하여 일치 항목을 출력할 수 있다. 

```javascript
Java.perform(function() {
  Java.enumerateLoadedClasses( {
    "onMatch": function(className) {
      console.log(className)
    },
    "onComplete": function() {}
  })
})
```

아래와 같이 CLI 환경에서 코드를 작성하여 후킹할 수 있지만 오타가 발생하면 긴 코드를 작성하기 까다로울 때가 있을 수 있다. 

![image-20240209202153264](/images/2024-02-09-android-frida-cli/image-20240209202153264.png)

따로 test.js 파일을 만들고 `-l` 옵션을 통해 스크립트 파일을 불러올 수 있다. `-U` 옵션과 같이 사용해준 후 js파일의 경로를 입력한 후 패키지명을 입력하면 된다.

![image-20240209202355097](/images/2024-02-09-android-frida-cli/image-20240209202355097.png)

![image-20240209202558258](/images/2024-02-09-android-frida-cli/image-20240209202558258.png)

<br>

### 안드로이드 액티비티 onResume() 함수 재작성

- 안드로이드 액티비티의 생명 주기는 `onCreate() -> onStart() -> onResume()` 함수를 실행하고 나서 액티비티가 포그라운드 실행상태가 된다. `onResume()` 함수는 해당 프로세스가 시작될 때 실행되거나 , 홈 화면 또는 다른 앱으로 전환했다가 다시 돌아오는 경우 실행된다.

- `onResume()` 함수가 실행될 때마다 로그가 출력되도록 작성하면 아래와 같다. 이 함수는 android.app.Activity 클래스에 존재한다. 

  ![image-20240209213437812](/images/2024-02-09-android-frida-cli/image-20240209213437812.png)

  ![image-20240209213552733](/images/2024-02-09-android-frida-cli/image-20240209213552733.png)

<br>

### 힙에서 인스턴스화된 객체 찾기

- `android.view.View`는 앱에서 가시적으로 보이는 모든 것들을 가리키는 클래스를 말한다. 

- `Java.choose` API를 사용하여 힙을 스캔한 뒤 View 클래스의 인스턴스를 찾으면 로그를 출력하는 예는 다음과 같다.

  ![image-20240209214943306](/images/2024-02-09-android-frida-cli/image-20240209214943306.png)

  ![image-20240209215001394](/images/2024-02-09-android-frida-cli/image-20240209215001394.png)

  

  

