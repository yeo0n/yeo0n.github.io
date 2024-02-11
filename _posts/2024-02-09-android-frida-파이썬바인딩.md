---
layout: single
title: "Frida 파이썬 바인딩"
date: 2024-02-09 21:51 +0900
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

## Frida 작업 자동화

- Frida CLI를 이용해서 자바스크립트를 직접 작성하거나, JS파일을 로드하여 후킹하는 방법을 알아보았었는데, 이번에는 파이썬을 이용하여 JS파일을 후킹하는 과정을 자동화 해보려고 한다.

<br>

### 파이썬 바인딩 과정

1.  프로세스 식별과 연결
2. 연결된 디바이스에서 구현하려는 프로세스와 세션 연결
3. 자바스크립트로 작성된 페이로드 자동 삽입

<br>

### 파이썬 바인딩 기본 구조

```python
# frida, sys 모듈 임포트
import frida, sys

# 삽입할 자바스크립트 코드 넣기
jscode = """payload_code"""

# frida를 시작하고 USB 장치에서 com.your.package.name 프로세스 연결
session = frida.get_usb_device().attach("com.your.package.name")

# jscode에 있는 스크립트 코드를 frida에서 사용할 수 있도록 생성
script = session.create_script(jscode)

# 생성한 script를 로드
script.load()

# script가 동작하기 전에 종료되는 문제 방지
sys.stdin.read()
```

<br>

```python
# frida, sys 모듈 임포트
import frdia, sys

# 삽입할 자바스크립트 코드 넣기
jscode = """payload_code"""

# frida를 시작하고 USB 장치에 연결
device = frida.get_usb_device()

# 연결된 USB 장치에서 com.your.package.name 프로세스 생성
pid = device.spawn(["com.your.package.name"])

# com.your.package.name 프로세스 연결
session = device.attach(pid)

# jscode에 있는 스크립트를 frida에서 사용할 수 있도록 생성
script = session.create_script(jscode)

# 생성한 script를 로드
script.load()

# com.your.package.name 프로세스 메인 스레드 실행
device.resume.(pid)

# script가 동작하기 전에 종료되는 문제 방지
sys.stdin.read()
```

<br>
이전에 android.app.Activity 클래스의 onResume 함수를 덮어쓰는 인젝션 코드를 파이썬 바인딩 파일로 만들면 아래와 같다.

```python
import frida, sys
jscode = """
console.log("[*] Starting onResume script");
Java.perform(function() {
    var Activity = Java.use("android.app.Activity");
    Activity.onResume.implementation = function() {
        console.log("[*] onResume() got called!");
        this.onResume();
    };
})
"""

session = frida.get_usb_device().attach("com.android.chrome")
script = session.create_script(jscode)
script.load()
sys.stdin.read()
```

![image-20240209222733229](/images/2024-02-09-android-frida-파이썬바인딩/image-20240209222733229.png)





