---
layout: single
title: "FridaLab 풀이1"
date: 2024-02-12 15:54 +0900
categories: 
    - Android
tag: 
#typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---

## FridaLab

> FridaLab은 Frida를 쉽게 연습할 수 있도록 Ross Marks가 만든 앱이며, 총 8개의 문제로 구성되어 있다. 아래 URL을 통해서 FridaLab apk 파일을 다운로드 받을 수 있다.

[FridaLab 설치](https://rossmarks.uk/blog/fridalab/)

<img src="/images/2024-02-09-android-fridalab/image-20240209224338173.png" alt="image-20240209224338173"  />

<br>

apk 파일을 다운로드 받은 후 apk 파일을 Nox 에뮬레이터에 드래그하면 자동으로 설치가 진행된다.

![image-20240209224419615](/images/2024-02-09-android-fridalab/image-20240209224419615.png)

<br>

FridaLab앱에 접속하면 아래와 같은 화면을 볼 수 있는데 가운데 CHECK 버튼을 통해 문제가 해결되었는지 알 수 있다.

![image-20240209224551260](/images/2024-02-09-android-fridalab/image-20240209224551260.png)

<br>`frida-ps -Ua` 명령을 통해 현재 실행 중인 앱의 패키지 명을 알 수 있다. 이를 통해 FridaLab의 패키지 명은 `uk.rossmarks.fridalab`인 것을 알 수 있다.

![image-20240209224816854](/images/2024-02-09-android-fridalab/image-20240209224816854.png)

<br>

### Challenge 01

1번 문제는 `challenge_01` 클래스의 변수인 `chall01`의 값을 1로 변경하면 된다. 

<br>jadx 도구를 이용해 소스코드를 확인하면 `getChall01Int` 함수를 통해 `chall01`을 리턴하고 있다.

![image-20240209230114224](/images/2024-02-09-android-fridalab/image-20240209230114224.png)

<br>

`getChall01Int` 함수를 우클릭 하여 frida 스니펫으로 복사를 통해 코드를 조금이라도 더 쉽게 작성하자!

![image-20240209230414427](/images/2024-02-09-android-fridalab/image-20240209230414427.png)

<br>

파이썬 바인딩을 통해 spawn으로 소스 코드를 작성하였다.  에뮬레이터가 느려질 경우, 프리다가 연결을 자동적으로 종료하는 걸 방지하기 위해 `setImmediate`를 사용하고 `Java.perform`다음 `chall01`의 value를 1로 변경한다.

```python
import frida, sys

jscode = """
setImmediate(function() {
    Java.perform(function() {
        var challenge_01 = Java.use("uk.rossmarks.fridalab.challenge_01");
        console.log("[*] chall01의 value를 1로 변경했습니다!")
        challenge_01.chall01.value = 1;
    });
});
"""

# frida를 시작하고 USB 장치에 연결
device = frida.get_usb_device()

# 연결된 USB 장치에서 com.package.name 프로세스 '생성' (이 때 메인 스레드는 시작되지 않은 상태여야 한다)
pid = device.spawn(["uk.rossmarks.fridalab"])

# com.package.name 프로세스 연결
session = device.attach(pid)

# jscode에 있는 스크립트 코드를 frida에서 사용할 수 있도로고 생성
script = session.create_script(jscode)

# 생성한 script를 로드 (앱의 메인 스레드가 실행 전인 상태이다)
script.load()

# com.package.name 프로세스 메인 스레드 실행
device.resume(pid)

# script가 동작하기 전에 종료되는 문제 예방
sys.stdin.read()

```

![image-20240209232522168](/images/2024-02-09-android-fridalab/image-20240209232522168.png)

<br>
그럼 아래와 같이 1번이 초록색으로 바뀐 것을 볼 수 있다. 

![image-20240210022608475](/images/2024-02-09-android-fridalab/image-20240210022608475.png)

<br>

### Challenge 02

2번 문제는 chall02의 메소드를 실행하라고 나와있다. jadx로 ctrl+shift+F 단축키를 통해 chall02를 검색하면 chall02의 메소드를 확인할 수 있다. 이는 MainActivity 클래스 내에 존재하는데 static 키워드가 없기 때문에 `instance` 메소드인 것을 알 수 있다. 이는 `Java.choose` 를 통해 가져올 수 있다.

![image-20240210022844904](/images/2024-02-09-android-fridalab/image-20240210022844904.png)

<br>

아래의 코드를 1번 푼 소스 코드 아래에 집어 넣어 1번과 2번이 같이 CHECK 되도록 해준다. 소스 코드를 설명하면 `Java.choose`함수를 이용해 인스턴스를 다룰 수 있다. 따라서 `chall02` 메소드는 MainActivity 클래스에 있으므로, 첫 번째 인자에 적어주고, 다음으로 `Java.choose` 함수는 `onMatch`와 `onComplete` 가 꼭 와야 한다. `onMatch`는 일치하는 인스턴스를 찾을 경우, chall02인스턴스를 찾은 후 호출하고, `onComplete` 를 통해 호출이 끝나면 challenge 02가 끝났다는 문자열을 출력하는 소스 코드이다.

```javascript
Java.choose("uk.rossmarks.fridalab.MainActivity", {
            "onMatch" : function(chall_02){
                chall_02.chall02();
            },
            "onComplete" : function(){
                console.log("[*] challenge 02 completed!")
            }
})
```

![image-20240210024014740](/images/2024-02-09-android-fridalab/image-20240210024014740.png)

![image-20240210024021137](/images/2024-02-09-android-fridalab/image-20240210024021137.png)

<br>

### Challenge 03

challenge 03은 `chall03` 메소드의 반환값을 true로 바꿔줘야 한다. jadx 도구로 텍스트 검색을 통해 chall03을 검색하면 아래와 같이 MainActivity 클래스 아래에 retrun으로 false를 반환하고 있다.

![image-20240210024313338](/images/2024-02-09-android-fridalab/image-20240210024313338.png)

<br>

`chall03` 메소드를 우클릭 후, fida 스니펫으로 복사하면, 약간 변경해주어야 하지만 코드를 작성하기 조금 더 수월하다. `.use` 함수를 통해 클래스를 먼저 선언해주고, `변경할 메소드.implementation` 을 이용하면 클래스에 정의된 메소드를 다 지우고 새로 작성하게 된다. 따라서 아래와 같이 completed 문자열을 출력하고 return 값을 true로 변경해주면 된다.

```javascript
let MainActivity = Java.use("uk.rossmarks.fridalab.MainActivity");
MainActivity.chall03.implementation = function () {
    console.log("[*] challenge 03 completed!!")
    return true;
};
```

![image-20240210025001982](/images/2024-02-09-android-fridalab/image-20240210025001982.png)
![image-20240210025009467](/images/2024-02-09-android-fridalab/image-20240210025009467.png)

<br>

### Challenge 04

`chall04` 메소드에 frida라는 문자열을 전달하라고 나와있다. jadx 도구로 확인하면 MainActivity 클래스에 아래와 같이 전달받은 인자의 문자열이 frida면 complete 되는 것을 알 수 있다.

![image-20240210025140158](/images/2024-02-09-android-fridalab/image-20240210025140158.png)

<br>

아래의 소스 코드를 3번째 소스 코드 아래에 붙여주고, chall04 메소드를 호출하고 인자를 넘겨주면 되기 때문에 `Java.choose` 함수를 사용해서 `chall04` 메소드의 인자를 `"frida"` 로 선언해주면 해결된다.

```javascript
Java.choose("uk.rossmarks.fridalab.MainActivity", {
    "onMatch": function(chall_04) {
        chall_04.chall04("frida");
    },
    "onComplete": function() {
        console.log("[*] challenge 04 completed!!!")
    } 
});
```

![image-20240210033043255](/images/2024-02-09-android-fridalab/image-20240210033043255.png)

![image-20240210033049535](/images/2024-02-09-android-fridalab/image-20240210033049535.png)

<br>

### Challenge 05

5번 째의 챌린지는 `chall05` 메소드에 "frida" 라는 문자열을 CHECK 버튼이 클릭될 때마다 항상 보내도록 해야 한다. jadx 도구로 검색할 필요 없이 이번에 바로 밑에 `chall05` 메소드의 소스 코드가 나와있다.

![image-20240210033411541](/images/2024-02-09-android-fridalab/image-20240210033411541.png)

<br>

CHECK버튼이 클릭될 때는 `implementation` 함수를 통해서 새로 정의하여 클릭될 때 "frida"가 항상 인자로 전달되도록 설정하면 된다. 여기서 주의할점은 위 챌린지에서 MainActivity 변수를 사용했으므로 MainActivity2라는 변수로 선언해서 작성해주면 해결할 수 있다.

```javascript
let MainActivity2 = Java.use("uk.rossmarks.fridalab.MainActivity");
MainActivity2.chall05.implementation = function(str) {
    this.chall05("frida");
    console.log("[*] challenge 05 completed!!!")
};
```

![image-20240210034347483](/images/2024-02-09-android-fridalab/image-20240210034347483.png)

![image-20240210034356575](/images/2024-02-09-android-fridalab/image-20240210034356575.png)

<br>

### Challenge 06

6번 문제는 올바른 값으로 10초 후 `chall06` 메소드를 호출해야 한다.  소스 코드는 아래와 같이 나와있고 아래 조건식에 confirmChall06(i)의 메소드를 확인해본다.

![image-20240210034550699](/images/2024-02-09-android-fridalab/image-20240210034550699.png)

<br>
challenge_06 클래스의 `confirmChall06` 함수는 boolean 인 것을 알 수 있고, 조건문을 확인해보면 i라는 변수가 chall06과 같아야 하고, 두 번쨰 조건은 startTime 메소드를 호출한 값을 timeStart에 저장하고 이 시간이 10초가 지나야 참이 된다. 

![image-20240210035140094](/images/2024-02-09-android-fridalab/image-20240210035140094.png)

<br>

먼저 아래와 같이 `setTimeout` 함수를 이용해 일정 시간이 지난 후 명령이 출력되도록 함수를 작성해줄 수 있다. 소스 코드를 실행시켜보면 10초 후에 아래의 문자열이 출력된다. 이는 콜백 함수를 감싸기 위해 `setImmediate`함수 외부에서 작성 해주어야 한다. 

```javascript
setTimeout(function() {
    console.log("[*] challenge 06 completed!!!")
}, 10000);
```

<br>
두 번째 조건을 만족했으니 첫 번째 조건인 `i == chall06`을 true로 설정해주고 호출하면된다. 따라서 `chall06` 메소드를 호출할 때 challenge_06 클래스의 confirmchall06 메소드를 수정해줌으로써 `chall06` 메소드의 인자를 아무 숫자나 넣어주면 해결된다.

```javascript
setTimeout(function() {
    Java.choose("uk.rossmarks.fridalab.MainActivity", {
        "onMatch": function(chall_06) {
            let challenge_06 = Java.use("uk.rossmarks.fridalab.challenge_06");
            challenge_06.confirmChall06.implementation = function () {
                return true;
            };
            chall_06.chall06(1); 
        },
        "onComplete": function() {
            console.log("[*] challenge 06 completed!!!");
        }
    });
}, 10000);
```

![image-20240210044908676](/images/2024-02-09-android-fridalab/image-20240210044908676.png)

![image-20240210044919904](/images/2024-02-09-android-fridalab/image-20240210044919904.png)

<br>

### Challenge 07

7번 문제는 check07Pin 메소드를 브루트포스하여 chall07 메소드를 확인해야 한다.  MainActivity 소스 부터 확인하면 문자열 형식의 인자를 받고 있다. 

![image-20240210045212858](/images/2024-02-09-android-fridalab/image-20240210045212858.png)

<br>

check07Pin 메소드를 살펴보면 str의 인자가 chall07과 같은지 비교하고 있고, chall07은 그 위에 랜덤한 값으로 정의되어 있는 것을 알 수 있다. 따라서 그냥 바로 check07Pin을 boolean 형식으로 되어 있으므로 return true로 변경해서 chall07 메소드를 호출하고 인자를 아무 문자열이나 넣어주면 해결된다.

![image-20240210045356287](/images/2024-02-09-android-fridalab/image-20240210045356287.png)

```javascript
Java.choose("uk.rossmarks.fridalab.MainActivity", {
    "onMatch": function(chall_07) {
        let challenge_07 = Java.use("uk.rossmarks.fridalab.challenge_07");
        challenge_07.check07Pin.implementation = function (str) {
            return true;
        };
        chall_07.chall07("frida");
    },
    "onComplete": function() {
        console.log("[*] challenge 07 completed!!!")
    } 
});

```

![image-20240210045933993](/images/2024-02-09-android-fridalab/image-20240210045933993.png)

![image-20240210045941422](/images/2024-02-09-android-fridalab/image-20240210045941422.png)

<br>

### Challenge 08

8번 문제는check라고 되어있는 버튼을 Confirm으로 변경해야 한다. MainActivity의 소스 코드는 아래와 같고 R.id.check의 텍스트 값이 Confirm이면 true가 반환된다.

![image-20240210050102518](/images/2024-02-09-android-fridalab/image-20240210050102518.png)

<br>
R.id.check를 확인해보면 16진수로 되어 있는 것을 알 수 있다. 

![image-20240210051823595](/images/2024-02-09-android-fridalab/image-20240210051823595.png)

<br>

또한 MainActivity 소스 코드의 import 부분에 `android.widget.Button`을 사용하는 것을 알 수 있다.

![image-20240212000529401](/images/2024-02-09-android-fridalab/image-20240212000529401.png)

<br>8번에서 풀이한 소스 코드는 아래와 같다. MainActivity클래스 내에서 버튼을 호출해야 하기 때문에 `Java.use`를 사용하여 `android.widget.Button`을 불러온다. 다음은 `findViewById` 함수가 위 코드에서 사용되었기 때문에 똑같이 이 함수를 사용해 `R.id.check`를 불러오고, `cast` 함수로 `findViewById` 함수로 찾은 id를 버튼이라고 명시적으로 지정해주어야 한다. 또한 string 변수를 통해서 문자열 객체를 선언하고, `stirng.$new`를 통해 `Confirm` 문자열을 `setText` 함수로 지정해주면 해결된다.

```javascript
Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) {
        let btnClass = Java.use('android.widget.Button');
        let id = instance.findViewById(0x7f07002f);
        let checkBtn = Java.cast(id, btnClass);
        let string = Java.use('java.lang.String');
        checkBtn.setText(string.$new('Confirm'));
      },
      onComplete: function() {
        console.log('Challenge 8 clear!');
      }
    });
```

<br>

클리어!

![image-20240212003702510](/images/2024-02-09-android-fridalab/image-20240212003702510.png)