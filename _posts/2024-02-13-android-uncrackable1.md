---
layout: single
title: "Uncrackable Level 1 풀이"
date: 2024-02-13 00:46 +0900
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

## Uncrackable Level 1

<br>

### Uncrackable.apk란? 

- Uncrackable.apk은 OWASP에서 제공하는 후킹 테스트용 앱으로 안드로이드 버전과 iOS 버전 모두 존재한다.
- 안드로이드 버전은 Level 3까지 있고, iOS 버전은 Level 2까지 있다.
- [Uncrackable-Level1.apk](https://github.com/OWASP/owasp-mastg/blob/master/Crackmes/Android/Level_01/UnCrackable-Level1.apk) 에서 apk 파일을 다운로드 받을 수 있다.



### 풀이

먼저 jadx 도구를 이용해서 디컴파일하고, AndroidManifest.xml 파일을 보면 아래와 같이 MainActivity 클래스의 액티비티가 존재한다는 것을 알 수 있다.

![image-20240213010239373](/images/2024-02-13-android-uncrackable1/image-20240213010239373.png)

<br>

Nox 에뮬레이터를 통해서 Uncrackable1.apk를 열어보면 루트가 탐지되었다는 메세지가 뜨면서 앱이 종료된다. 따라서 frida를 통해 루팅 탐지 우회를 성공하여야 한다. 

![image-20240213005049578](/images/2024-02-13-android-uncrackable1/image-20240213005049578.png)

<br>

MainActivity의 소스 코드를 분석해보면 onCreate를 찾을 수 있는데 onCreate 함수는 앱이 실행될 때 가장 먼저 실행되는 영역이다. 또한 루트가 탐지되는 소스 코드라는 것도 알 수 있는데, `c.a, c.b, c.c`의 메소드에 의해서 루트가 탐지되고 있다.

![image-20240213012806623](/images/2024-02-13-android-uncrackable1/image-20240213012806623.png)

<br>c를 더블클릭하면 c 클래스의 a,b,c 메소드를 확인할 수 있다.  a는 PATH라는 환경변수 값을 가져오고, 만약 PATH 값에 su라는 문자열이 존재하면 true를 반환한다. b는 Build.TAGS라는 안드로이드 태그에 "test-keys" 문자열이 존재한다면 true를 반환한다. c는 루팅에 쓰이는 Superuser.apk, daemonsu 등이 존재하는지 확인하여 true를 반환한다.

![image-20240213013001735](/images/2024-02-13-android-uncrackable1/image-20240213013001735.png)

<br>
이제 frida를 통해서 루팅 탐지를 우회하여야 한다. 먼저 c라는 클래스의 a, b, c의 반환값을 전부 false로 지정하는 방법과, a 메소드의 System.exit(0);을 지우는 방법이 있다. 나는 a,b,c라는 메소드의 반환값을 false로 지정하는 방법을 선택했다.

<br>
frida를 사용하기 위해서 Nox 에뮬레이터 adb 쉘에서 frida 서버를 실행시켜준 후, `frida-ls-devices` 명령을 통해 디바이스와 연결 되었는지 확인한다. 다음으로 Uncrackable1.apk를 실행시키고 `frida-ps -Ua` 명령으로 현재 실행 중인 애플리케이션 PID와 패키지 이름을 확인할 수 있다.

![image-20240213015438137](/images/2024-02-13-android-uncrackable1/image-20240213015438137.png)

<br>

frida를 통해 후킹하는 방법은 여러 가지가 존재한다. CLI 환경에서 직접 소스 코드를 작성하여도 되지만 코드가 길거나 잘못 작성할 경우 조금 불편하고, js 파일을 직접 작성하고 `-ㅣ` 옵션을 통해 js 파일을 로드하여 후킹할 수 있다. 그리고 또 다른 방법으로는 파이썬 바인딩을 이용해 후킹을 할 수 있지만 이번에는 js 파일을 frida에 로드하여 후킹하는 방법을 사용하려고 한다.

<br>

먼저 js 파일을 하나 생성하여 아래와 같이 코드를 작성해준다. perform 함수를 통해 디바이스에 연결되었는지 확인하고, `Java.use` 함수를 통해 c 클래스를 가져와 a,b,c 메소드를 false를 리턴하도록 재정의 해준다.

![image-20240213024403726](/images/2024-02-13-android-uncrackable1/image-20240213024403726.png)

<br>
아래와 같이 `frida -U -f owasp.mstg.uncrackable1` 을 통해 apk 파일을 spawn 형식으로 먼저 띄우지 않고 열어주고, `-l` 옵션을 통해 후킹할 js 코드의 경로를 지정해주면 된다.

![image-20240213024735257](/images/2024-02-13-android-uncrackable1/image-20240213024735257.png)

<br>

그럼 Root Detected 문자열이 보이지 않는 것을 알 수 있다. 

![image-20240213024857297](/images/2024-02-13-android-uncrackable1/image-20240213024857297.png)

<br>
이제 입력할 문자열에 test를 넣고 VERIFY 버튼을 클릭하면 Nope 문자열이 보인다. 이 소스 코드를 분석해보자.

![image-20240213024946223](/images/2024-02-13-android-uncrackable1/image-20240213024946223.png)

<br>jadx 도구로 해당 문자열을 검색해도 되고 MainActivity 소스 코드를 분석해도 된다. 그럼 아래 조건인 obj의 텍스트를 가져와 `a.a(obj)` 값이 true이면 Success 문자열이 뜬다는 것을 알 수 있다. 여기서 obj는 해당 액티비티에서 입력한 값을 가져오는 것이므로 a.a 메소드만 분석하면 되겠다.

![image-20240213025054850](/images/2024-02-13-android-uncrackable1/image-20240213025054850.png)

<br>

a.a의 소스 코드를 분석해보면, `8d127684cbc37c17616d806cf50473cc`와 `5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=`를 base64로 디코딩한 값을 a.a.a 메소의 인자에 넣어주고 있다. 

```java
public class a {
    public static boolean a(String str) {
        byte[] bArr;
        byte[] bArr2 = new byte[0];
        try {
            bArr = sg.vantagepoint.a.a.a(b("8d127684cbc37c17616d806cf50473cc"), Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0));
        } catch (Exception e) {
            Log.d("CodeCheck", "AES error:" + e.getMessage());
            bArr = bArr2;
        }
        return str.equals(new String(bArr));
    }

```

<br>
a.a.a 메소드는 아래와 같고, AES를 복호화해주는 코드이다. 

![image-20240213032242558](/images/2024-02-13-android-uncrackable1/image-20240213032242558.png)

<br>

아래 js 코드에서 a.a.a의 인자에 어떤 값들이 있는지 출력해주는 소스 코드를 작성하여 후킹해주면 실패하지만 어떤 값들이 a.a.a의 인자로 들어가는지 알 수 있다.

```javascript
let a = Java.use("sg.vantagepoint.a.a");
a.a.implementation = function (bArr, bArr2) {
    console.log("bArr= ", bArr);
    console.log("bArr2= ", bArr2);
};  
```

<br>
실행해보면 bArr과 bArr2에 어떤 값이 들어있는지 알 수 있다.

![image-20240213033610449](/images/2024-02-13-android-uncrackable1/image-20240213033610449.png)

<br>
이제 나온 인자들을 그대로 bArr과 bArr2에 리스트 형식으로 넣어준 후, a 함수를 실행하면 어떤 값이 나오는지 알아보자.

```javascript
let a = Java.use("sg.vantagepoint.a.a");
a.a.implementation = function (bArr, bArr2) {
    var bArr = [-115,18,118,-124,-53,-61,124,23,97,109,-128,108,-11,4,115,-52];
    var bArr2 = [-27,66,98,21,-53,91,-102,6,-61,-96,-75,-26,-92,-67,118,-102,73,-24,-16,116,-8,46,-1,29,-107,-85,124,23,20,118,24,-25];
    console.log(a.a(bArr, bArr2));
};  
```

<br>
그럼 이 값들을 아스키 코드로 변환해주어야 한다. 

![image-20240213034142063](/images/2024-02-13-android-uncrackable1/image-20240213034142063.png)

<br>
result 변수에 나온 문자열의 길이 만큼 `String.fromCharCode` 함수를 이용해 아스키코드를 문자열로 변환해서 출력하면 된다.

```javascript
let a = Java.use("sg.vantagepoint.a.a");
a.a.implementation = function (bArr, bArr2) {
    var bArr = [-115,18,118,-124,-53,-61,124,23,97,109,-128,108,-11,4,115,-52];
    var bArr2 = [-27,66,98,21,-53,91,-102,6,-61,-96,-75,-26,-92,-67,118,-102,73,-24,-16,116,-8,46,-1,29,-107,-85,124,23,20,118,24,-25];
    var flag = "";
    var result = a.a(bArr, bArr2);
    var i;
    for(i=0; i<result.length; i++){
        flag += String.fromCharCode(result[i]);
    }
    console.log(flag);

};  
```

![image-20240213035455409](/images/2024-02-13-android-uncrackable1/image-20240213035455409.png)

<br>
`I want to believe` 문자열을 넣어주기 위해 루트 탐지 코드를 제외하고 전부 지운 후 실행하여 문자열을 넣어주면 Success 성공!

![image-20240213040911168](/images/2024-02-13-android-uncrackable1/image-20240213040911168.png)