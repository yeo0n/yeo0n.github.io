---
layout: single
title: "Uncrackable Level 2 풀이"
date: 2024-02-13 15:23 +0900
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

## Uncrackable-Level2

[Uncrackable-Level2.apk](https://github.com/OWASP/owasp-mastg/blob/master/Crackmes/Android/Level_02/UnCrackable-Level2.apk) 파일을 다운로드 받고 Nox 에뮬레이터로 드래그 앤 드롭을 하여 apk 파일을 설치해준다. 앱을 실행하면 Level1과 같이 루트가 탐지되었다는 메세지가 보인다.

![image-20240213152440515](/images/2024-02-13-android-uncrackable2/image-20240213152440515.png)

<br>

jadx 도구로 디컴파일하여 Root detected! 메세지 나온 소스 코드를 분석해보면 onCreate 함수에 b 클래스의 a,b,c 메소드가 true로 반환될 경우 Root detected! 메세지가 나오는 것을 알 수 있다. 

![image-20240213152805879](/images/2024-02-13-android-uncrackable2/image-20240213152805879.png)

<br>
이번 level2 에서는 파이썬 바인딩을 이용해 후킹을 진행할 것이다. 먼저 adb 쉘에서 frida 서버를 열어준 후, `frida-ps -Uai` 명령을 통해 현재 설치된 앱의 패키지 명을 알 수 있다. Uncrackable Level 2의 패키지 이름은 owasp.mstg.uncrackable2 이다.

![image-20240213153448019](/images/2024-02-13-android-uncrackable2/image-20240213153448019.png)

<br>
파이썬 바인딩을 이용해 후킹을 하기 위해 기본 뼈대부터 작성해주자.

![image-20240213153544921](/images/2024-02-13-android-uncrackable2/image-20240213153544921.png)

<br>

b 클래스의 a,b,c 메소드를 분석해보니 Level1과 동일하게 작성되어 있다. a 메소드는 PATH 환경 변수에 su 문자열이 있는지 검사하고, b 메소드는 태그 검사, c 메소드는 루팅에 사용되는 앱들을 검사하고 있다.

![image-20240213154107940](/images/2024-02-13-android-uncrackable2/image-20240213154107940.png)

<br>

jscode 안에 `sg.vantagepoint.a.b` 클래스를 지정하고 a,b,c 메소드의 return 값을 전부 false로 변경하는 소스 코드를 작성했다.

![image-20240213160447789](/images/2024-02-13-android-uncrackable2/image-20240213160447789.png)

<br>

CLI 환경에서 `frida -U -f owasp.mstg.uncrackable2 --pause` 를 통해 Uncrackable2 앱을 실행하고 pause로 멈춰놓는다.

![image-20240213160636419](/images/2024-02-13-android-uncrackable2/image-20240213160636419.png)

<br>
이제 파이썬 소스 코드를 실행하고 frida CLI에서 %resume 명령을 통해 앱의 실행을 재개해주면 아래와 같이 루트 탐지를 우회한 것을 볼 수 있다.

![image-20240213160813326](/images/2024-02-13-android-uncrackable2/image-20240213160813326.png)

![image-20240213160828715](/images/2024-02-13-android-uncrackable2/image-20240213160828715.png)

![image-20240213160836455](/images/2024-02-13-android-uncrackable2/image-20240213160836455.png)

<br>

Level1과 마찬가지로 VERIFY 버튼을 클릭하여 success 메세지가 나와야 한다. 

![image-20240213160954812](/images/2024-02-13-android-uncrackable2/image-20240213160954812.png)

<br>
Nope라는 문자열의 소스 코드는 아래와 같고, obj라는 변수에 사용자가 입력한 값을 가져온 후, 이 값이 `this.m.a(obj)`라는 값과 일치하면 Success라는 메세지를 출력한다.

![image-20240213161048099](/images/2024-02-13-android-uncrackable2/image-20240213161048099.png)

<br>

m은 메인액티비티 소스 코드의 상단에 private로 Codecheck 클래스를 m이라는 변수로 지정해주고 있는 것을 알 수 있다.

![image-20240213161258793](/images/2024-02-13-android-uncrackable2/image-20240213161258793.png)

<br>m.a 메소드인 CodeCheck 클래스의 a 메소드를 살펴보면 `bar(str.getBytes());`를 통해 return 값을 반환하고 있다. `getBytes` 메소드는 str인 문자열을 default charset으로 인코딩하여 byte 배열로 반환해준다. 즉, 입력받은 문자열을 72, 101, 108 이런식으로 만들어준다. bar 메소드를 살펴보면 native로 지정되어 있어 so 파일을 분석해야 한다.

![image-20240213161342776](/images/2024-02-13-android-uncrackable2/image-20240213161342776.png)

<br>

jadx 도구로는 native로 작성된 메소드를 확인할 수 없다. 따라서 IDA를 이용해 so 파일을 import 해온 후 분석해야 하는데 apktool을 이용해 디컴파일 하여 so 파일을 얻어야 한다. 

![image-20240213163825366](/images/2024-02-13-android-uncrackable2/image-20240213163825366.png)

<br>

이제 IDA를 열어서 디컴파일 폴더의 lib -> x86 폴더 안에 있는 so 파일을 open해서 f5 버튼으로 디컴파일을 해준다. `Thanks for all the fish` 라는 문자열과 비교하고 있다.

![image-20240213165451459](/images/2024-02-13-android-uncrackable2/image-20240213165451459.png)

<br>

위 문자열을 그대로 입력하고 VERIFY 버튼을 누르면 Success 메세지가 나오게 된다.

![image-20240213165603947](/images/2024-02-13-android-uncrackable2/image-20240213165603947.png)

